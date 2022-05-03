---
layout: post
title:  "Movendo o /tmp para a Memoria Ram"
date:   20/03/2022 18:00:00 -0300
mdate:   03/05/2022 20:00:00 -0300
tags:   linux ram tmp
---

### Usando Memoria Ram para arquivos temporários

Para computadores com bastante memoria ram, e com SSD, é recomendado montar o /tmp na ram, para isso edite o arquivo /etc/fstab:

Caso o seu /tmp esteja montado em uma partição separada, comente a linha no arquivo /etc/fstab e adicione a linha abaixo para usar 2Gb de ram para o diretorio /tmp:

~~~bash
tmpfs /tmp tmpfs defaults,noatime,nosuid,nodev,mode=1777,size=2048M 0 0
~~~

## Referencias

[fstab](https://venkateshshukla.wordpress.com/2014/10/20/fstab-auto-mount-hide-various-drives-on-your-system-with-example/)