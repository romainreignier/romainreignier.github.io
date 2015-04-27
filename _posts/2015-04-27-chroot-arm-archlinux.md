---
layout: post
title : "Chroot pour une autre architecture ArchlinuxARM"
date:   2015-04-27 22:00:00
categories: linux
tags :
- linux
- raspberrypi
- archlinux
- arm
---

## Contexte

[![Logo TBB]({{ site.baseurl }}/images/logo_tbb.png "Logo TBB")](https://www.threadingbuildingblocks.org/)

Pour compiler [OpenCV](http://opencv.org/) avec le support d'intel [TBB](https://www.threadingbuildingblocks.org/) pour la parallélisation sur le [Raspberry Pi 2](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/), j'ai dû télécharger les sources de TBB, de le compiler et d'ajouter moi-même les fichiers dans `/usr/include` et `/usr/lib` (voir [mon article sur le sujet]({% post_url 2015-04-19-installer-opencv3-raspberrypi2 %}). Cependant, pour conserver un système [Archlinux](https://www.archlinux.org/) propre, il vaut mieux installer tout nouveau paquet avec le gestionnaire de paquets [`pacman`](https://wiki.archlinux.org/index.php/Pacman) et le construire avec l'outil [`makepkg`](https://wiki.archlinux.org/index.php/Makepkg) qui prend en entrée un fichier [`PKGBUILD`](https://wiki.archlinux.org/index.php/PKGBUILD).

[![Logo Archlinux ARM]({{ site.baseurl }}/images/logo_alarm.png "Logo Archlinux ARM")](http://archlinuxarm.org/)

Mais une fois le PKGBUILD réalisé et testé avec succès, j'ai décidé de le proposer à la communauté [ArchlinuxARM](http://archlinuxarm.org/) en proposant un [Pull Request sur le Github du dépôt officiel](https://github.com/archlinuxarm/PKGBUILDs/pull/1175). Dans mon PKGBUILD, je n'avais autorisé que la compilation pour l'architecture `armv7h`, mais lors de la souscription, on m'a demandé de justifier cette restriction. C'était simplement puisque je n'avais pas testé sur d'autres architectures, ce qu'on m'a donc demandé de faire.

[![Logo ARM]({{ site.baseurl }}/images/logo_arm.gif "Logo ARM")](http://arm.com/products/processors/instruction-set-architectures/index.php)

Problème, les autres architectures supportées par ArchlinuxARM sont `armv5` et `armv6h`. Pour la dernière, j'ai bien un [Raspberry Pi de première génération](https://www.raspberrypi.org/products/model-b/), mais rien pour l'`armv5`. Il me faut donc utiliser l'outil [`chroot`](https://wiki.archlinux.org/index.php/Change_root) comme conseillé par mon correspondant sur Github. Seulement, je ne sais pas comment l'utiliser pour une architecture différente de la mienne et je ne trouve pas documentation claire sur le sujet.

## Chroot sur une architecture différente
L'idée est donc de faire un `chroot` sur un système d'une architecture différente que l'architecture du système hôte qui sera ici un Odroid C1 donc `armv7h`. On reste donc quand même entre systèmes ARM.

- Installation des outils nécessaires avec les scripts Archlinux `arch-install-scripts` qui vont bien, tout est dans le paquet `devtools-arlarm` (sources : [un post du forum ArchlinuxARM](http://archlinuxarm.org/forum/viewtopic.php?f=60&t=8770&p=46540&hilit=chroot#p46540) et [Developper Wiki](https://wiki.archlinux.org/index.php/DeveloperWiki:Building_in_a_Clean_Chroot)) :

		$ sudo pacman -S devtools-alarm

- Maintenant que nous avons les outils pour le `chroot`, il nous faut un système cible avec l'architecture `armv5` que l'on peut trouver sur le site [ArchlinuxARM rubrique armv5](http://archlinuxarm.org/platforms/armv5). On choisi alors une carte de développement pour accéder à l'url du tarball pour le télécharger avec `wget` que l'on installe si on ne l'a pas déjà (je partais d'une distribution fraîchement installée pour l'occasion) :

		$ sudo pacman -S wget
		$ wget http://archlinuxarm.org/os/ArchLinuxARM-armv5-latest.tar.gz

- Une fois en possession du fichier, on arrête de rigoler et on passe superutilisateur (pas sûr à 100% que cela soit obligatoire) :		

		$ sudo su

- On crée un dossier pour accueillir notre système hôte :

		# mkdir chroot

- Et maintenant on extrait le tarball dans ce dossier (commande trouvée dans le [guide d'installation sur l'OLinuxIno](http://archlinuxarm.org/platforms/armv5/olinuxino)):

		# bsdtar -xpf ArchLinuxARM-armv5-latest.tar.gz -C chroot/

- Il est maintenant temps de passer en `chroot` dans ce dossier en utilisant les scripts Archlinux qui simplifient la vie (évitent de monter les bonnes partitions en mode `--bind` à la main) :

		# arch-chroot chroot/ /bin/bash

- Une fois dans le nouveau système, on se met bien en mettant à jour les paquets :

		# pacman -Syu

- Et en installant le minimum vital :

		# pacman -S vim-minimal
		# pacman -S base-devel

- La commande pour construire un paquet à partir d'un `PKGBUILD`, `makepkg` n'accepte pas d'être exécutée en tant que root, il faut alors créer un nouvel utilisateur :

		# useradd -m -g users -G wheel -s /bin/bash romain
		# passwd romain

- Puis donner les droits de `sudo` à cet utilisateur :

		# visudo

	Et décommenter la ligne :

		%wheel ALL=(ALL) ALL

- Voilà, il ne reste plus qu'à passer sur le compte de l'utilisateur (source : [wiki de smhteam.info](http://smhteam.info/wiki/index.linux.php5?wiki=ChrooterUnUtilisateur)):

		# su - romain

- Maintenant, on peut tranquillement construire notre paquet avec `makepkg`.

Finalement, `intel-tbb` ne peut être compilé que sur `armv7h` donc j'ai fait tout ça pour rien. Mais maintenant je sais le faire !

Merci à `WarheadsSE` du channel IRC `#archlinux-arm` sur Freenode pour le coup de main.