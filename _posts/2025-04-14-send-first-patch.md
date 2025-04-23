---
title: Kernel Contribution - Sending our first contribution
categories: [Open Source Software Development, Kernel Contribuitions, MAC0470]
tags: [linux, kernal, refactoring, patch, iio, open-source, kernel-linux]
render_with_liquid: false
---
Collaborators [Gabriel Lima](https://gabriellimmaa.github.io/), [Gabriel José](https://gabrielpereir4.github.io/gabriel-portfolio/), [Vitor Marques](https://vitormarquesr.github.io/blog/).

After we solve [Kernel Contribution - Refactoring hardware initialization for VEML6030 and VEML6035 sensors](https://gabriellimmaa.github.io/posts/refactoring-common-hardware-initialization-in-VEML603x-drivers-(veml6030-&-veml6035)/). We sent the patch about our contribution.

The patch is sent by email. We send the changes to the course monitors and receive their feedback on the patches, some minor changes in the description and organization of the patches only.

This was the feedback from the monitors on our first patch:

```
### COMENTÁRIOS ###

1) A mensagem de commit está muito boa (talvez valha a pena só refinar a
primeira frase com um preâmbulo, como, "The functions veml6030...").
Porém, o prefixo do título do commit parece estar um pouco incorreto. Ao
invés de

iio: light : veml6030 Remove code duplication

deveria ser

iio: light: veml6030: Remove code duplication

2) Há problemas de coding style no seu patch, tanto na mensagem de
commit quanto no conteúdo do patch. Para detectar estes problemas, rode

git format-patch -1 --stdout | ./scripts/checkpatch.pl --

MPORTANTE: As coisas apontadas pelo `checkpatch.pl` não são 100%
confiáveis, então avaliem se o que ele sugere é razoável;

3) Um destaque no corpo da mensagem de commit, percebi que instruí vocês
de forma incorreta (vou até atualizar no pad). O `Co-developed-by`,
aparentemente, deve ser seguido por um `Signed-off-by`. Então, ao final
do patch, estas tags tem que ser

Signed-off-by: Vitor Marques <vitor.marques@ime.usp.br>
Co-developed-by: Gabriel Lima <gabriellimamoraes@ime.usp.br>
Signed-off-by: Gabriel Lima <gabriellimamoraes@ime.usp.br>
Co-developed-by: Gabriel José <gabrieljpe@ime.usp.br>
Signed-off-by: Gabriel José <gabrieljpe@ime.usp.br>

ou seja, exceto o committer (a pessoa que rodou `git commit ...`), cada
um deve ter primeiro o `Co-developed-by` seguido de `Signed-off-by`;

4) Sobre o mérito da remoção da duplicação em si, sempre reduzir
duplicação faz sentido, mas quem irá determinar a validade deste caso
neste contexto serão os mantenedores e a comunidade.
```

The main issue is that we had forgotten to run 'checkpatch' before sending the patch, thus the coding style problems.
After the feedback from the TAs, we ran 'checkpatch' on the patch and the coding style problems detected were in the following
categories:

* Trailing whitespaces
* Identation errors, mainly in conditional `if` clauses
* Brackets {} placed after the function definition instead of below it
* Lines exceeding 100 characters

Finally, after making the corrections, we sent it again to the monitors and the official Kernel maintainers.
