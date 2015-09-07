---
layout: post
title : "Utiliser la ST Nucleo avec mbed hors ligne"
date:   2015-09-01 11:14:00
categories: linux
tags :
- linux
- arm
- archlinux
- ubuntu
---
## Ubuntu

### Introduction
[Mbed](https://developer.mbed.org/) est une plateforme de développement pour les microcontrôleurs ARM Cortex-M. Elle propose un compilateur en ligne avec un framework pour faciliter la programmation et la protabilité sur plusieurs plateforme. Le compilateur en ligne permet d'éviter d'installer une toolchain sur son PC mais impose un accès permanent à internet. Il est possible d'exporter les projets de certaines plateformes sous la forme d'un projet pour GNU GCC à base de Makefile. C'est le cas pour les cartes de développement ST Nucleo (testé avec la F030, F401, F411).

### Ubuntu 15.04
Avec Ubuntu Vivid 15.04, il suffit d'installer la chaîne de compilation pour processeurs ARM *none*, c'est à dire sans OS ou *bare-metal*.

    $ sudo apt install gcc-arm-none-eabi

Elle devrait également installer :

    binutils-arm-none-eabi
    libnewlib-arm-none-eabi
    libstdc++-arm-none-eabi-newlib

Si ce n'est pas le cas, installer le paquet manquant.

### Ubuntu 14.04
Avec la version LTS de Ubuntu en cours, Ubuntu Trusty 14.04, il manque le paquet `libstdc++-arm-none-eabi-newlib`.

Pour l'installer, soit on l'installe manuellement en téléchargeant l'archive `.deb`. Voir [cet article](http://josho.org/blog/blog/2014/11/30/nucleo-gcc/) pour la démarche.

Ou alors, on utilise un [PPA](https://launchpad.net/~terry.guo/+archive/ubuntu/gcc-arm-embedded) qui fournit toute la chaîne de compilation pour ARM (utilisée généralement avant que les paquets ne soient disponibles dans les dépôts officiels).

Dans ce cas, la procédure est décrite sur la page du PPA. Il faut commancer par s'assurer que la version des dépôts officiels n'est pas installée pour ne pas rencontrer de conflits :

    $ sudo apt-get remove binutils-arm-none-eabi gcc-arm-none-eabi
    $ sudo add-apt-repository ppa:terry.guo/gcc-arm-embedded
    $ sudo apt-get update
    $ sudo apt-get install gcc-arm-none-eabi=4.9.3.2015q1-0trusty13

### Exporter un projet Mbed
Dans le compilateur en ligne, choisir d'exporter le projet au format **GCC (ARM Embedded)**. Une fois l'archive téléchargée et décompressée, il suffit alors de faire un `make` dans le terminal. Ensuite, comme pour avec le compilateur en ligne, copier le fichier `.bin` dans le support amovible apparu lors du branchement de la carte Nucleo par USB.

### Utilisation du JTAG/SWD

#### OpenOCD
Pour utiliser l'interface de debogage JTAG/SWD, nous avons besoin d'OpenOCD. La version dans les dépôts ne comprend pas les définitions pour les cartes Nucleo, nous allons donc le compiler à partir des sources.

Vérifier que les dépendances nécessaires sont bien installées

    $ sudo apt-get install libtool automake libusb-1.0.0-dev

Téléchargement des sources sur Github :

    $ git clone https://github.com/ntfreak/openocd

Compilation et installation :

    $ cd openocd
    $ ./bootstrap
    $ ./configure
    $ make
    $ sudo make install

#### STLink
On installe maintenant un programme permettant d'accéder à l'interface de programmation STLink de ST.

    $ git clone https://github.com/texane/stlink
    $ cd stlink
    $ ./autogen.sh
    $ ./configure
    $ make
    $ sudo make install

Le programme STLink vient avec des règles `udev` à installer.

    $ sudo cp 49-stlinkv2-1.rules /etc/udev/rules.d/
    $ sudo service udev restart

#### Flash du programme

##### Connexion avec OpenOCD
On lance OpenOCD avec le fichier de configuration correspondant à notre carte Nucleo.

    $ openocd -f /usr/local/share/openocd/scripts/board/st_nucleo_f4.cfg

Dans un autre terminal, on va lancer une session tellnet sur le port 4444 ouvert pour l'occasion par OpenOCD (il ouvre aussi le port 3333 pour le debug avec GDB).

    $ telnet localhost 4444

Une fois dans la console tellnet, on va exécuter 3 commandes pour stopper le micorocontrôleur, envoyer le programme, puis le relancer :

    > resel halt
    > flash write_image erase Nucleo_blink_led.hex
    > reset run

Pour quitter tellnet :

    > exit

##### Script de programmation OpenOCD
Il est un peu rébarbatif d'exécuter les commandes précédentes. Heureusement, il exite un script pour ça ([doc](http://openocd.org/doc/html/Flash-Programming.html)) :

    $ openocd -f /usr/local/share/openocd/scripts/board/st_nucleo_f4.cfg -c "program NucleoF401_blink_led.elf verify reset exit"

Que l'on peut très bien ajouter à la fin de notre Makefile :

    openocd:
    	openocd -f /usr/local/share/openocd/scripts/board/st_nucleo_f4.cfg -c "program $(PROJECT).elf verify reset exit"

Ainsi, il ne reste plus qu'à utiliser la commande :

    $ make openocd

## Archlinux
La manipulation fonctionne aussi sous Archlinux, sauf que tout est déjà dans les dépôts ;)

La toolchain :

    $ yaourt -S arm-none-eabi-binutils arm-none-eabi-gcc arm-none-eabi-gdb arm-none-eabi-newlib

Openocd :

    $ yaourt -S openocd

Pour StLink, il faut prendre la version Git dans AUR car la dernière release de stlink ne prend pas en charge stlink-v2_1 :

    $ yaourt -S stlink-git

Enfin pour le script d'openocd, il n'est pas placé au même endroit :

    /usr/share/openocd/scripts/board/st_nucleo_f4.cfg

    openocd:
    	openocd -f /usr/share/openocd/scripts/board/st_nucleo_f4.cfg -c "program $(PROJECT).elf verify reset exit"

Et voilà !
