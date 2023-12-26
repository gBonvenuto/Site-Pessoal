---
title: "Impressão Digital Para Login"
date: 2023-12-25T21:27:50-03:00
author: Giancarlo Bonvenuto
tags: ["Login", "Fingerprint"]
categories: ["Linux", "Debian"]
description: "Como eu configurei a impressão digital para funcionar como Login (inclusive para comandos como sudo)"
keywords: "Linux, login, Debian, fingerprint"
draft: false
includeToc: true
localCss: []
externalCss: []
externalCssDownload: []
localJs: []
externalJs: []
externalJsDownload: []
useMath: false
---

## Instalação

Esse artigo não é um tutorial, mas um "diário" de como eu fiz para configurar a impressão digital no Linux.

Primeiramente, eu segui [este tutorial](https://wiki.debian.org/SecurityManagement/fingerprint%20authentication). Contudo, eu uso Debian testing, e infelizmente tanto `fprintd` quanto `libpam-fprintd` não estavam disponível para instalação. Assim tive que baixar diretamente do site do Debian os pacotes .deb em [fprintd](https://packages.debian.org/bookworm/fprintd) e [libpam-fprintd](https://packages.debian.org/unstable/libpam-fprintd).

## Registrando a digital

Para registrar o indicador direito, basta executei o seguinte comando:

```fish
fprintd-enroll $USER
```
*Em que `$USER` é o usuário ao qual a impressão digital pertence.*

*Alem do mais, caso não funcione de primeira, tente executar como `sudo`.*

E fiquei colocando e tirando o dedo do sensor até que aparecesse a mensagem `Enroll result: enroll-completed`

Para adicionar outros dedos, como o indicador esquerdo, eu utilizei:

```fish
fprintd-enroll $USER -f left-index-finger
```

E para ver o nome de cada dedo, eu fiz `fprintd-enroll $USER -f a`, em que "a" não é um dedo válido, assim ele listará todos os dedos disponíveis

## Utilização para comandos como sudo

Em seguida habilitei a impressão digital para comandos como `sudo`:

> # [Fingerprint Authentication with sudo](https://wiki.debian.org/SecurityManagement/fingerprint%20authentication)
> 
> To enable fingerprint authentication with sudo, run
> 
> ```
> pam-auth-update
> ```
> 
> and enable the "Fingerprint authentication" profile by checking the corresponding checkbox and then pressing "OK" (see Screenshot pam-auth-update). 

## Utilizando para logar no sistema

Enfim, para habilitar o `fprintd` no meu login manager ([ly](https://github.com/fairyglade/ly)), adicionei as seguintes linhas:

```
auth 	   sufficient   pam_unix.so try_first_pass likeauth nullok
auth	   sufficient   pam_fprintd.so
```

Assim, o arquivo `/etc/pam.d/ly` ficou:
```
#%PAM-1.0

auth 	   sufficient   pam_unix.so try_first_pass likeauth nullok
auth	   sufficient   pam_fprintd.so
auth       include      login
account    include      login
password   include      login
session    include      login
```

### Utilizando com `i3lock`

Para travar o meu notebook quando eu fecho a tela, eu utilizo o [i3lock-color](https://github.com/Raymo111/i3lock-color). Assim, para habilitar a leitura de impressão digital também no i3lock eu adicionei as mesmas linhas:

```
auth 	   sufficient   pam_unix.so try_first_pass likeauth nullok
auth	   sufficient   pam_fprintd.so
```

E assim, o arquivo `/etc/pam.d/i3lock` ficou:
```
#
# PAM configuration file for the i3lock-color screen locker. By default, it includes
# the 'system-local-login' configuration file (see /etc/pam.d/system-local-login)
# for Arch and Gentoo and 'login' for Debian. Note that upstream uses only 'login',
# which doesn't work on Arch and Gentoo.
#

#auth include system-local-login # For Arch/Gentoo
auth 	 sufficient   pam_unix.so try_first_pass likeauth nullok
auth	 sufficient   pam_fprintd.so
auth include login # For Debian
```

### Para outros sistemas de login como o `gddm`

Ainda não testei, mas pretendo algum dia criar uma máquina virtual e ver se consigo. Isso porque passei um ano inteiro utilizando [Pop!_OS](https://pop.system76.com/) com Gnome, e não fazia ideia de que era possível habilitar a leitura de impressão digital.

Mas caso você tenha curiosidade em habilitar no seu gdm ou outro login manager, eu recomendo uma leitura na [ArchWiki](https://wiki.archlinux.org/title/fprint)

## Problemas

Para a utilização com o comando `sudo`, a leitura de impressão digital funcional muito bem. Contudo para o `ly` e o `i3lock` eu preciso inserir uma senha errada (ou vazia) antes de eles conseguirem detectar minha impressão digital.

Este não é um problema muito grave, pois é só um clique de `Enter` a mais. Mas de qualquer forma continuarei procurando alguma solução para tal problema
