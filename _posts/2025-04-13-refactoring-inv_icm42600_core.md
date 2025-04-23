---
title: Kernel Contribution - Refactoring inv_icm42600_core.c functions
categories: [Open Source Software Development, Kernel Contribuitions, MAC0470]
tags: [linux, kernal, refactoring, patch, iio, open-source, kernel-linux]
render_with_liquid: false
---
This refactoring had as collaborators [Gabriel Lima](https://gabriellimmaa.github.io/), [Gabriel José](https://gabrielpereir4.github.io/gabriel-portfolio/), [Vitor Marques](https://vitormarquesr.github.io/blog/).

### Objective

Remove duplicated code between the hardware initialization functions inv_icm42600_set_accel_con and inv_icm42600_set_gyro_conf

Both functions shared around  **90% identical logic** .

---

### Options Considered

| Option      | Description                                                                                                      |
| ----------- | ---------------------------------------------------------------------------------------------------------------- |
| **1** | ✅ Componentize shared code blocks into unique functions.                         |
| **2** | Rewrite code blocks in a way new componentized functions could be created.		 |

We chose **Option 1** as most code blocks in both functions had unique parameters and aspects. In that way, we chose to componentize fully shared blocks.

---

### The Final Function: `xnv_icm42600_common_init`

```c
/**
* xnv_icm42600_common_init - Common config function initialization for icm_42600
* @oldconf: Pointer to the existing or previous configuration structure. Used as a fallback for any unset values.
* @conf: Pointer to the new configuration structure. Fields with values < 0 will be overwritten using oldconf.
*
* This function performs a basic initialization routine for the inv_icm42600 sensor configuration. It sanitizes the new configuration*  * (conf) by filling in any unset fields (i.e., fields with values less than 0) using corresponding values from the previous or current  * configuration (oldconf).
* It ensures that no configuration field remains in an invalid state before proceeding with further setup or sensor initialization.
*
* This function has no return statement.
*/

void xnv_icm42600_common_init(struct inv_icm42600_sensor_conf *oldconf, struct inv_icm42600_sensor_conf *conf){
    /* Sanitize missing values with current values */
    if (conf->mode < 0)
    conf->mode = oldconf->mode;
    if (conf->fs < 0)
    conf->fs = oldconf->fs;
    if (conf->odr < 0)
    conf->odr = oldconf->odr;
    if (conf->filter < 0)
    conf->filter = oldconf->filter;
}
```

---

### Applying It to Original Functions

#### `inv_icm42600_set_accel_conf`

```c
int inv_icm42600_set_accel_conf(struct inv_icm42600_state *st,
				struct inv_icm42600_sensor_conf *conf,
				unsigned int *sleep_ms)
{
	struct inv_icm42600_sensor_conf *oldconf = &st->conf.accel;
	unsigned int val;
	int ret;

	xnv_icm42600_common_init(oldconf, conf);

	/* force power mode against ODR when sensor is on */
	switch (conf->mode) {
	case INV_ICM42600_SENSOR_MODE_LOW_POWER:
	case INV_ICM42600_SENSOR_MODE_LOW_NOISE:
		if (conf->odr <= INV_ICM42600_ODR_1KHZ_LN) {
			conf->mode = INV_ICM42600_SENSOR_MODE_LOW_NOISE;
			conf->filter = INV_ICM42600_FILTER_BW_ODR_DIV_2;
		} else if (conf->odr >= INV_ICM42600_ODR_6_25HZ_LP &&
			   conf->odr <= INV_ICM42600_ODR_1_5625HZ_LP) {
			conf->mode = INV_ICM42600_SENSOR_MODE_LOW_POWER;
			conf->filter = INV_ICM42600_FILTER_AVG_16X;
		}
		break;
	default:
		break;
	}

	/* set ACCEL_CONFIG0 register (accel fullscale & odr) */
	if (conf->fs != oldconf->fs || conf->odr != oldconf->odr) {
		val = INV_ICM42600_ACCEL_CONFIG0_FS(conf->fs) |
		      INV_ICM42600_ACCEL_CONFIG0_ODR(conf->odr);
		ret = regmap_write(st->map, INV_ICM42600_REG_ACCEL_CONFIG0, val);
		if (ret)
			return ret;
		oldconf->fs = conf->fs;
		oldconf->odr = conf->odr;
	}

	/* set GYRO_ACCEL_CONFIG0 register (accel filter) */
	if (conf->filter != oldconf->filter) {
		val = INV_ICM42600_GYRO_ACCEL_CONFIG0_ACCEL_FILT(conf->filter) |
		      INV_ICM42600_GYRO_ACCEL_CONFIG0_GYRO_FILT(st->conf.gyro.filter);
		ret = regmap_write(st->map, INV_ICM42600_REG_GYRO_ACCEL_CONFIG0, val);
		if (ret)
			return ret;
		oldconf->filter = conf->filter;
	}

	/* set PWR_MGMT0 register (accel sensor mode) */
	return inv_icm42600_set_pwr_mgmt0(st, st->conf.gyro.mode, conf->mode,
					  st->conf.temp_en, sleep_ms);
}
```

#### `inv_icm42600_set_gyro_conf`

```c
int inv_icm42600_set_gyro_conf(struct inv_icm42600_state *st,
			       struct inv_icm42600_sensor_conf *conf,
			       unsigned int *sleep_ms)
{
	struct inv_icm42600_sensor_conf *oldconf = &st->conf.gyro;
	unsigned int val;
	int ret;

	xnv_icm42600_common_init(oldconf, conf);

	/* set GYRO_CONFIG0 register (gyro fullscale & odr) */
	if (conf->fs != oldconf->fs || conf->odr != oldconf->odr) {
		val = INV_ICM42600_GYRO_CONFIG0_FS(conf->fs) |
		      INV_ICM42600_GYRO_CONFIG0_ODR(conf->odr);
		ret = regmap_write(st->map, INV_ICM42600_REG_GYRO_CONFIG0, val);
		if (ret)
			return ret;
		oldconf->fs = conf->fs;
		oldconf->odr = conf->odr;
	}

	/* set GYRO_ACCEL_CONFIG0 register (gyro filter) */
	if (conf->filter != oldconf->filter) {
		val = INV_ICM42600_GYRO_ACCEL_CONFIG0_ACCEL_FILT(st->conf.accel.filter) |
		      INV_ICM42600_GYRO_ACCEL_CONFIG0_GYRO_FILT(conf->filter);
		ret = regmap_write(st->map, INV_ICM42600_REG_GYRO_ACCEL_CONFIG0, val);
		if (ret)
			return ret;
		oldconf->filter = conf->filter;
	}

	/* set PWR_MGMT0 register (gyro sensor mode) */
	return inv_icm42600_set_pwr_mgmt0(st, conf->mode, st->conf.accel.mode,
					  st->conf.temp_en, sleep_ms);

	return 0;
}
```

---

### Conclusion

This change componentized a fully duplicated initialization code into a single function, eliminating redundancy and promoting better maintenance. Centralizing common code blocks into single functions promotes a better code quality, avoiding the task of writing the same code twice and as such, avoiding possible errors in such task.
