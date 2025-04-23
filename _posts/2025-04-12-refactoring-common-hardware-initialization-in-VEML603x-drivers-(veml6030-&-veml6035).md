---
title: Kernel Contribution - Refactoring hardware initialization for VEML6030 and VEML6035 sensors
categories: [Open Source Software Development, Kernel Contribuitions, MAC0470]
tags: [linux, kernal, refactoring, patch, iio, open-source, kernel-linux]
render_with_liquid: false
---
This refactoring had as collaborators [Gabriel Lima](https://gabriellimmaa.github.io/), [Gabriel José](https://gabrielpereir4.github.io/gabriel-portfolio/), [Vitor Marques](https://vitormarquesr.github.io/blog/).

### Objective

Remove duplicated code between the hardware initialization functions of the ambient light sensors **VEML6030** and  **VEML6035** , located in:

* `light/veml6030.c::veml6030_hw_init`
* `light/veml6030.c::veml6035_hw_init`

Both functions shared around  **90% identical logic** .

---

### Options Considered

| Option      | Description                                                                                                      |
| ----------- | ---------------------------------------------------------------------------------------------------------------- |
| **1** | Extract shared blocks into small helper functions while keeping separate init functions.                         |
| **2** | ✅ Create a**fully parameterized function** that centralizes all logic and abstracts only the differences. |

We chose **Option 2** for being cleaner, more consistent, and easier to maintain.

---

### The Final Function: `veml603x_hw_common_init`

```c
/**
* veml603x_hw_common_init - Common hardware initialization for VEML sensors
* @indio_dev: IIO device structure
* @dev: Device structure
* @iio_init_val1: Initial value for IIO GTS configuration
* @iio_init_val2: Secondary initial value for IIO GTS configuration
* @gain_sel: Array of gain selection pairs
* @gain_sel_size: Size of the gain selection array
* @it_sel: Array of integration time selection multipliers
* @it_sel_size: Size of the integration time selection array
* @als_conf_val2: Configuration value for ALS (Ambient Light Sensor)
*
* This function performs the common initialization steps for VEML sensors,
* including configuring gain, integration time, power saving mode, and
* thresholds. It also ensures the sensor is properly powered on and ready
* for operation.
*
* Returns 0 on success or a negative error code on failure.
*/

int veml603x_hw_common_init(
struct iio_dev *indio_dev,
struct device *dev,
int iio_init_val1,
int iio_init_val2,
const struct iio_gain_sel_pair *gain_sel,
size_t gain_sel_size,
const struct iio_itime_sel_mul *it_sel,
size_t it_sel_size,
int als_conf_val2
)
{
	int ret, val;
	struct veml6030_data *data = iio_priv(indio_dev);
	  
	ret = devm_iio_init_iio_gts(dev, iio_init_val1, iio_init_val2,
			gain_sel, gain_sel_size,
			it_sel, it_sel_size,
			&data->gts);

	if (ret)
	return dev_err_probe(dev, ret, "failed to init iio gts\n");
	  
	ret = veml6030_als_shut_down(data);
	if (ret)
	return dev_err_probe(dev, ret, "can't shutdown als\n");

	ret = regmap_write(data->regmap, VEML6030_REG_ALS_CONF, als_conf_val2);
	if (ret)
	return dev_err_probe(dev, ret, "can't setup als configs\n");

	ret = regmap_update_bits(data->regmap, VEML6030_REG_ALS_PSM, VEML6030_PSM | VEML6030_PSM_EN, 0x03);
	if (ret)
	return dev_err_probe(dev, ret, "can't setup default PSM\n");

	ret = regmap_write(data->regmap, VEML6030_REG_ALS_WH, 0xFFFF);
	if (ret)
	return dev_err_probe(dev, ret, "can't setup high threshold\n");

	ret = regmap_write(data->regmap, VEML6030_REG_ALS_WL, 0x0000);
	if (ret)
	return dev_err_probe(dev, ret, "can't setup low threshold\n");

	ret = veml6030_als_pwr_on(data);
	if (ret)
	return dev_err_probe(dev, ret, "can't poweron als\n");

	ret = devm_add_action_or_reset(dev, veml6030_als_shut_down_action, data);
	if (ret < 0)
	return ret;

	/* Clear stale interrupt status bits if any during start */
	ret = regmap_read(data->regmap, VEML6030_REG_ALS_INT, &val);
	if (ret < 0)
	return dev_err_probe(dev, ret,
	"can't clear als interrupt status\n");
	  
	return ret;
}
```

---

### Applying It to Original Drivers

#### `veml6035_hw_init`

```c
static int veml6035_hw_init(struct iio_dev *indio_dev, struct device *dev)
{
	veml603x_hw_common_init(
		indio_dev,
		dev,
		0,
		409600000,
		veml6035_gain_sel,
		ARRAY_SIZE(veml6035_gain_sel),
		veml6030_it_sel,
		ARRAY_SIZE(veml6030_it_sel),
		VEML6035_SENS | VEML6035_CHAN_EN | VEML6030_ALS_SD
	);
	return 0;
}
```

#### `veml6030_hw_init`

```c
static int veml6030_hw_init(struct iio_dev *indio_dev, struct device *dev)
{
	return veml603x_hw_common_init(
		indio_dev,
		dev,
		2,
		150400000,
		veml6030_gain_sel,
		ARRAY_SIZE(veml6030_gain_sel),
		veml6030_it_sel,
		ARRAY_SIZE(veml6030_it_sel),
		0x1001
	);
}
```

---

### Parameter Differences Between Devices

| Parameter         | VEML6030              | VEML6035                         |
| ----------------- | --------------------- | -------------------------------- |
| `iio_init_val1` | `2`                 | `0`                            |
| `iio_init_val2` | `150400000`         | `409600000`                    |
| `gain_sel`      | `veml6030_gain_sel` | `veml6035_gain_sel`            |
| `it_sel`        | `veml6030_it_sel`   | `veml6030_it_sel` *(shared)* |
| `als_conf_val2` | `0x1001`            | `VEML6035_SENS`                |

---

### Conclusion

This change eliminated more than 100 lines of duplicated code, reduced future maintenance costs, and simplified the process of adding support for other VEML sensors in the future. Centralizing logic via a single parameterized function has made the driver code much more consistent and scalable.
