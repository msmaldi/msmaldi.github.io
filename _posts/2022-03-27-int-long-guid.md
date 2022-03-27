---
layout: post
title:  "Int, Long, Guid."
date:   27/03/2022 18:00:00 -0300
tags: programming dotnet c#
---

### Comparação entre os tipos de 32, 64 e 128 bits

Qual usar no banco de dados?

Um resumo sobre o tamanho:

Type        | Tamanho             | Minimo                     | Maximo
:----------:|:-------------------:|:--------------------------:|:------------------------:
int, Int32  | 32 bits <br /> (4 bytes)   |             -2.147.483.648 |             2.147.483.647
long, Int64 | 64 bits <br /> (8 bytes)   | -9.223.372.036.854.775.808 | 9.223.372.036.854.775.807
System.Guid | 128 bits <br /> (16 bytes) | 00000000-0000-0000 <br /> 0000-000000000000 | FFFFFFFF-FFFF-FFFF <br /> FFFF-FFFFFFFFFFFF                              

Olhando esses dados, quanto tempo demoraria para usar todos os indices?

Primeiros veremos com um Inteiro de 32 bits.

Imagine que inserimos 100 por segundos, então temos 22 milhões de segundos, ou 366.667 minutos, ou 6112 horas, ou 260 dias.

Agora com Inteiro de 64 bits.

Imagine que vc desenvolveu um aplicativo que vai rodar por 1000 anos, quantas inserções por segundo ele suportaria?

Considere então 1000 anos * 365 dias * 24 horas * 60 minutos * 60 segundos temos = 31.536.000.000, 31,5 bilhões.

Logo 9.223.372.036.854.775.807 / 31.536.000.000 = 292.471.208,67 inserções por segundos.

Agora imagine que salvaremos apenas o Id no banco de dados e nada mais, como o Id tem seu tamanho de 64bit ou 8 bytes, teriamos mais de 2GB por segundos.

Assim, o tipo long é suficiente para maioria dos casos.
