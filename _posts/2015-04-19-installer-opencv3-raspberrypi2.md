---
layout: post
title : "Installer OpenCV 3 Béta sur le RaspberryPi 2"
date:   2015-04-19 22:30:00
categories: linux
tags :
- linux
- raspberrypi
- archlinux
- ubuntu
- raspbian
- opencv
---

## Introduction

[![Logo OpenCV]({{ site.baseurl }}/images/logo_opencv.png "Logo OpenCV")](http://opencv.org/)

[OpenCV](http://opencv.org/) est une bibliothèque open-source de traitement d'image.

Le traitement d'image demande beaucoup de ressources matérielles. Pour optimiser les performances, il est possible d'activer la compilation avec [Threading Buildong Blocks](https://www.threadingbuildingblocks.org/) (TBB) qui est une biliothèque faite par Intel pour faciliter les calculs en parrallèle.

De plus, OpenCV 3 (en version béta à l'heure actuelle) permet l'utilisation des instructions [Neon](http://www.arm.com/products/processors/technologies/neon.php) sur les processeurs ARM. Néon est une unité de type SIMD d'ARM pour accélérer les calculs de type DSP ([Wikipédia](https://fr.wikipedia.org/wiki/ARM_NEON).

L'objectif est alors de compiler OpenCV 3 Béta avec le support de TBB et des instructions Néon pour le [Raspberry Pi 2](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/) (ARMv7).

## Raspbian

[![Logo Raspbian]({{ site.baseurl }}/images/logo_raspbian.png "Logo Raspbian")](http://www.raspbian.org/)

L'OS par défaut des Raspberry Pi est [Raspbian](http://www.raspbian.org/). Raspbian est basé sur Debian Wheezie et sur cette distribution, seul OpenCV 2.3 est disponible dans les dépôt avec des paquets comme `libopencv-core2.3`. Cette version pose problème car ne fonctionne pas avec le module caméra [Camera Module](https://www.raspberrypi.org/products/camera-module/) en utilisant le driver `v4l2` qui est façonb la plus standart d'accéder à la webcam dans Linux. En effet, le module du noyau `bcm2835-v4l2` permet de faire apparaître le module caméra dans `/dev/video0`, afin qu'OpenCV puisse y accéder.

D'après des témoignages sur le forums, cela fonctionne avec les dernières versions d'OpenCV 2.4.9 et 2.4.10, mais il faut alors les compiler à partir des sources. Quitte à compiler, autant utiliser la dernière version, même en béta, pour profiter des optimisations NEON. Problème, la version de Raspbian actuellement distribuée fonctionne à la fois sur le Raspberry Pi 1 (ARMv6) et le Raspberry Pi 2 (ARMv7) donc les instructions spécifiques à l'ARMv7 ne sont pas activées. Neon ne peut donc pas être utilisé, il faut se tourner vers une autre distribution.

## Ubuntu

[![Logo Ubuntu]({{ site.baseurl }}/images/logo_ubuntu.png "Logo Ubuntu")](https://wiki.ubuntu.com/ARM/RaspberryPi)

Il existe une [version de Ubuntu 14.04 LTS disponible](https://wiki.ubuntu.com/ARM/RaspberryPi) pour le Raspberry Pi 2. L'avantage d'Ubuntu, c'st qu'on peut facilement y installer [ROS](http://www.ros.org/) si besoin.

Il s'agit d'une configuration minimale à laquelle il faut ajouter une un environnement graphique si on le souhaite.

Pour Ubuntu, la version d'OpenCV disponible dans les dépôts est la 2.4.8. Mais il y a toujours un problème avec la gestion du module caméra avec v4l2.

Il est également possible de compiler OpenCV. Pour TBB, il est disponible directement dans les dépôts. Et pour Néon, Ubuntu ARM étant prévu pour ARMv7, il est activé.

Après plusieurs essais, la compilation fonctionne, mais impossible d'exécuter un programme OpenCV, il y a un problème d'affichage (GTK 2). Pour ne pas passer par GTK, j'installe Qt et recompile avec Qt. Mais cette fois, l'installation de Qt a créé des conflits dans la base d'`apt-get` car certains paquets graphiques tentent de remplacer ceux qui sont spécifiques au Raspberry Pi ("unmet dependencies").

Pour ne pas me prendre la tête plus longtemps, je décide d'utiliser une distibution que je connais bien, [Archlinux ARM](http://archlinuxarm.org/).

## ArchLinux

[![Logo Archlinux ARM]({{ site.baseurl }}/images/logo_alarm.png "Logo Archlinux ARM")](http://archlinuxarm.org/)

OpenCV 3 est disponible sur [AUR](https://aur.archlinux.org/) sous le nom de [opencv-git](https://aur.archlinux.org/packages/opencv-git/).

En revenant au plus simple, un Archlinux de base, on tente d'installer OpenCV.

Il y a donc une version d'Archlinux ARM pour Raspberry Pi 2 [ici](http://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2).

### Installation du système
On suit les instructions pour préparer la carte SD avec l'outil linux `fdisk`.

Avec l'utilisateur `root` et le mot de passe `root`, on peut se connecter, même en SSH.

- Mise à jour du système :

        # pacman -Syu

- Installation des paquets de base :

        # pacman -S base-devel

- Quelques utilitaires habituels :

        # pacman -S vim htop screen tree picocom git rsync

- Ajout d'un utilisateur :

        # useradd -m -g users -G wheel -s /bin/bash romain

- Mot de passe :

        # passwd romain

- Ajout à quelques groupes :

        # gpasswd -a romain video
        # gpasswd -a romain audio
        # gpasswd -a romain power
        # gpasswd -a romain uucp
        # gpasswd -a romain dialout

- Donner les droits sudo à l'utilisateur :

        # visudo

    Et décommenter la ligne suivante :

        %wheel ALL=(ALL) ALL

- Générer les locales en éditant le fichier :

        # vim /etc/locale.gen

    Décommenter la locale souhaitée (ici : `en_US.UTF-8` pour éviter tout conflit avec des langues utilisant des caractères spéciaux comme les accents en français).

    Générer les locales avec :

        # locale-gen

- Pour le clavier AZERTY, créer le fichier :

        # vim /etc/vconsole.conf

    Et y écrire :

        KEYMAP=fr-pc

- Réglage du fuseau horaire :

        # rm /etc/localtime
        # ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime

- Installation de `avahi-daemon` pour faciliter l'accès SSH par le nom de la machine :

        # pacman -S avahi nss-mdns
        # systemctl enable avahi-daemon.service
        # systemctl start avahi-daemon.service
             
- Création d'un espace swap (mieux pour compiler OpenCV après les problèmes rencontrés sous Ubuntu). Ici, on crée un fichier swap, c'est plus simple une fois que les partitions sont déjà crées ([source](https://wiki.archlinux.org/index.php/Swap) :

        #  fallocate -l 1G /swapfile
        #  chmod 600 /swapfile
        #  mkswap /swapfile
        #  swapon /swapfile

    Édition du fichier `fstab` pour monter le fichier de swap au démarrage :
    
        #  vim /etc/fstab

    Et ajout de la ligne :

        /swapfile none swap defaults 0 0

    On peut vérifier que la swap est bien présente :

        # free

- Redémarrage et connexion au compte utilisateur fraichement créé.

- Installation manuelle de `yaourt` (le dépôt `archlinuxfr` ne fonctionnait pas au moement de l'installation) [optionel] :

        $ mkdir Downloads
        $ cd Downloads/
        $ curl -O https://aur.archlinux.org/packages/pa/package-query/package-query.tar.gz
        $ tar zxvf package-query.tar.gz
        $ cd package-query
        $ makepkg -si
        $ cd ..
        $ curl -O https://aur.archlinux.org/packages/ya/yaourt/yaourt.tar.gz
        $ tar zxvf yaourt.tar.gz 
        $ cd yaourt
        $ makepkg -si

### Installation d'un environnement graphique

[![Logo i3]({{ site.baseurl }}/images/logo_i3.png "Logo i3")](http://i3wm.org/)

- Installation d'utilitaires pour le son :

        $ sudo pacman -S alsa-utils alsa-firmware alsa-lib alsa-plugins

- Installation de X11 :

        $ sudo pacman -S xorg-server xorg-xinit xorg-utils xorg-server-utils

- Installation du driver vidéo :

        $ sudo pacman -S xf86-video-fbdev

- Installation d'un environnement de bureau léger, [i3](http://i3wm.org/). Pour cela, on installe le groupe `i3` qui va installer les paquets `i3-wm`, `i3-status` et `i3-lock` :

        $ yaourt -S i3

- Installation des logiciels utiles, le menu lanceur d'applications `dmenu`, un terminal `rxvt-unicode` et un navigateur `uzbl-browser` :

        $ yaourt -S dmenu urxvt rxvt-unicode uzbl-browser

- Installation de polices :

        $ sudo pacman -S ttf-liberation ttf-bitstream-vera ttf-dejavu ttf-freefont

- Pour qu'`i3` se lance avec la commande `startx`, il faut ajouter la ligne suivante au fichier `~/.xinitrc` :

        exec i3

### Activer le X11 forwarding
Par défaut, le X11 forwarding est désactivé, générant une erreur de ce type lorsque l'on souhaite exécuter un programme avec GUI depuis une liaison ssh (`ssh -Y`) :

    Gtk-WARNING **: cannot open display:

Pour cela, éditer le fichier `/etc/ssh/sshd_config` :

    $ sudo vim /etc/ssh/sshd_config

Et ajouter les lignes suivantes (ou décommenter dans le fichier) :

    X11Forwarding yes
    X11UseLocalhost yes

### Compilation de TBB

[![Logo TBB]({{ site.baseurl }}/images/logo_tbb.png "Logo TBB")](https://www.threadingbuildingblocks.org/)

*Remarque : dans la philosophie Archlinux, il faudrait créer un PKGBUILD pour réaliser les actions suivantes, mais je ne sais pas faire.*

- Téléchargement des sources :

        $ cd ~/Downloads
        $ curl -O https://www.threadingbuildingblocks.org/sites/default/files/software_releases/source/tbb43_20150316oss_src.tgz
        $ tar zxvf tbb43_20150316oss_src.tgz
        $ cd Downloads/tbb43_20150316oss/

- Compilation de tbb :

        $ make CXXFLAGS="-DTBB_USE_GCC_BUILTINS=1 -D__TBB_64BIT_ATOMICS=0"

    Il faut ajouter les flags à la commande `make` parce qu'on est sur ARM ([source](https://software.intel.com/en-us/forums/topic/500680)). La commande `make` par défaut compile `tbb` et `tbbmalloc`. 

- Installation manuelle :

        $ sudo cp -r include/tbb /usr/include/
        $ cd build/linux_armv7_gcc_cc4.9.2_libc2.20_kernel3.18.11_release/
        $ sudo cp *.so* /usr/lib/
        $ cd ../linux_armv7_gcc_cc4.9.2_libc2.20_kernel3.18.11_debug/
        $ sudo cp *.so* /usr/lib/

### Compilation d'OpenCV
Ici, on prend donc `opencv-git` de AUR en modifiant le PKGBUILD selon nos besoins.

- Téléchargement des fichiers sources :

        $ curl -O https://aur.archlinux.org/packages/op/opencv-git/opencv-git.tar.gz
        $ tar zxvf opencv-git.tar.gz 
        $ cd opencv-git

- Édition du PKGBUILD :

        $ vim PKGBUILD 

    - Ajout de l'architecture `armv7h` :

            -- arch=('i686' 'x86_64')
            ++ arch=('i686' 'x86_64' 'armv7h')

    - Suppression de la dépendance à `intel-tbb` car la version dans les dépôts ne fonctionne pas sur ARM :

            -- depends=('gstreamer0.10-base' 'intel-tbb' 'openexr' 'xine-lib' 'libdc1394' 'gtkglext')
            ++ depends=('gstreamer0.10-base' 'openexr' 'xine-lib' 'libdc1394' 'gtkglext')

    - Modification des flags de compilation CMAKE :

            ++ '-D ENABLE_NEON=ON'
            ++ '-D TBB_INCLUDE_DIR:PATH=/home/romain/Downloads/tbb43_20150316oss/include'
            ++ '-D TBB_LIB_DIR:PATH=/home/romain/Downloads/tbb43_20150316oss/build/linux_armv7_gcc_cc4.9.2_libc2.20_kernel3.18.11_release'

        *Remarque : les fichiers TBB n'avaient pas encore été placés dans /usr/...les 2 dernières lignes sont peut-être inutiles.*  
        [Source](http://stackoverflow.com/questions/7994261/how-do-i-build-opencv-with-tbb)

    - Compilation parallèle pour utiliser les 4 coeurs du Raspberry Pi 2 :

            -- make
            ++ make -j4

- Création du paquet :

        $ makepkg -s

    *Remarque: cette étape dure environ 1 heure puiqu'elle effectue la compilation d'OpenCV.*

- Installation du paquet :

        $ sudo pacman -U opencv-git-3.0.0.beta.r975.g46a6f70-1-armv7h.pkg.tar.xz 

### Configuration du module camera
- Il faut modifier le fichier `/boot/config` pour activer le module caméra ([source](https://wiki.archlinux.org/index.php/Raspberry_Pi#Raspberry_Pi_camera_module)) :

        $ sudo vim /boot/config.txt

    Et y ajouter :

        gpu_mem=128
        start_file=start_x.elf
        fixup_file=fixup_x.dat

- Charger le module `bcm2835-v4l2` pour voir la caméra dans `/dev/video0` :

        sudo modprobe bcm2835-v4l2

    Pour qu'il soit chargé automatiquement au démarrage, ajouter le fichier suivant :

        $ sudo vim /etc/modules-load.d/rpi-camera.conf 

    Et y écrire :

        bcm2835-v4l2

Voilà, OpenCV devrait fonctionner avec le module caméra.

### Code de test
Testons un code tout simple pour vérifier que l'installation est bien fonctionnelle.

Voici le code du fichier `displayWebcam.cpp` qui affiche la webcam dans une fenêtre et s'arrete lorqu'on appuie sur la touche `Echap` :

{% highlight cpp %}
#include "opencv2/opencv.hpp"

using namespace cv;

int main() {
    VideoCapture cap(0);
    if(!cap.isOpened())
        return -1;
    namedWindow("Webcam",1);
    for(;;) {
        Mat frame;
        cap >> frame;
        imshow("Webcam", frame);
        if(waitKey(27) >= 0)
            break;
    }
    return 0;
}
{% endhighlight %}

Pour la génération du `Makefile`, on utilise `cmake`. Voici le contenu du `CMakeLists.txt` :

{% highlight cmake %}
cmake_minimum_required(VERSION 2.8)
project( DisplayWebcam )
find_package( OpenCV REQUIRED )
add_executable( DisplayWebcam displayWebcam.cpp )
target_link_libraries( DisplayWebcam ${OpenCV_LIBS} )
{% endhighlight %}

Ensuite, génération du `Makefile` :

    $ cmake .

Compilation :

    $ make

Test :

    $ ./DisplayWebcam
