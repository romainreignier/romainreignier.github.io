---
layout: post
title : Utiliser les boutons supplémentaires de sa souris sous Linux 
---

## Contexte

![Photo de la souris M500]({{ site.baseurl}}/images/m500.png "Photo de la souris M500")

J'ai acheté il y a peu, une nouvelle souris USB, la [Logitech M500](http://www.logitech.fr/fr-fr/product/corded-mouse-m500). J'en suis très satisfait, mais elle possède des boutons que l'on peut actionner en poussant la roulette et à droite et ceux-ci ne sont pas utilisés par défaut.

Que ce la soit dans Gnome ou Xfce, il n'y a pas d'option pour personnaliser les boutons de la souris dans les paramètres de la souris.

Étant donné que j'adore les onglets, je les collectionne, que ça soit dans Firefox ou le terminal, j'ai l'intention d'utiliser ces deux boutons supplémentaires pour naviguer entre mes onglets.

## Configuration
Il nous faut donc utiliser des outils qui vont agir directement sur le serveur X.

Tout d'abord, pour connaître le numéro de ces boutons, nous allons utiliser l'utilitaire [`xev`](http://www.xfree86.org/current/xev.1.html). Après l'avoir installé avec notre gestionnaire de paquets favoris ([`yaourt`](https://wiki.archlinux.fr/Yaourt) dans mon cas), il suffit de le lancer dans un terminal pour voir une petite fenêtre blanche apparaître. Il faut alors placer le curseur dans cette fenêtre et observer la sortie dans le terminal lorsque l'on actionne les boutons que l'on souhaite personnaliser :

    ButtonRelease event, serial 37, synthetic NO, window 0x3800001,
        root 0xa4, subw 0x0, time 70851433, (148,73), root:(742,907),
        state 0x10, button 6, same_screen YES

    ButtonPress event, serial 37, synthetic NO, window 0x3800001,
        root 0xa4, subw 0x0, time 70852001, (148,73), root:(742,907),
        state 0x10, button 7, same_screen YES

On repère ainsi qu'un appui sur la molette vers la gauche correspond au bouton 6 et vers la droite, c'est le bouton 7.

Forts de cette découverte, il nous faut maintenant trouver un moyen de lier un raccourcis clavier à ces deux boutons.

Une fois encore, c'est le fabuleux [wiki Archlinux](https://wiki.archlinux.org/index.php/All_Mouse_Buttons_Working#xvkbd_and_xbindkeys) qui nous sauve. La [page sur l'utilisation des boutons de la souris](https://wiki.archlinux.org/index.php/All_Mouse_Buttons_Working#xvkbd_and_xbindkeys) n'est pas très claire, avec de nombreuses méthodes différentes et des avertissements **Out of date** un peu partout qui ne rassurent pas trop. Mais bon, la méthode avec [`xvkbd`](http://homepage3.nifty.com/tsato/xvkbd/) et [`xbindkeys`](http://www.nongnu.org/xbindkeys/xbindkeys.html) a bien fonctionné pour moi.

Le principe est d'utiliser le clavier virtuel `xvkbd` pour générer les raccourcis clavier que l'on souhaite effectuer, `Ctrl + PageUp` et `Ctrl + PageDown` dans mon cas. La commande `xvkbd` sera elle appelée par `xbindkeys` qui gère les raccourcis clavier et des périphériques.

Alors la configuration n'est pas très *user-friendly* car elle repose sur des fichiers de configuration et des outils X un peu vieillots.

Tout d'abord on installe ces deux utilitaires :

    $ yaourt -S xvkbd xbindkeys

Puis on crée un fichier de configuration pour `xbindkeys` dans notre `$HOME` :

    $ vim ~/.xbindkeysrc 

Et on y place les lignes suivantes :

    "xvkbd  -text "\[Control]\[Page_Up]""
           m:0x0 + b:6

    "xvkbd  -text "\[Control]\[Page_Down]""
           m:0x0 + b:7

Dans le principe, la première ligne correspond à la commande à effectuer et la seconde est le racourcis utilisé. Ici on appelle `xvkbd` avec l'arguement `-text` auquel on passe les touches souhaitées. Je n'ai pas trouvé de documentation expliquant le nommage des touches, c'est après pluiseurs essais que je suis arrivé à une configuration fonctionnelle. Sur la deuxième ligne, on indique `m:0x0` pour indiquer que c'est la souris 0 qu'on utilise, suivi du numéro du bouton précédé par `b:` pour indiquer que c'est un bouton.

Remarque, il existe une interface graphique `xbindkeys` (dépendant de gtk 1, oh c'est vieux...) mais celle-ci ne capte pas les boutons de la souris donc inutile pour nous. Pour info, on peut la trouver sur [aur](https://aur.archlinux.org/packages/xbindkeys_config).

Il ne reste plus qu'à lancer `xbindkeys` pour tester la configuration :

    $ xbinkeys

Pour qu'il se lance avec la session, il faut ajouter la ligne suivante au fichier `~/.xinitrc` :

    xbindkeys

Un utilisateur de KDE est également passé par `xbindkeys` pour configurer une Logitech M500 [ici](http://blog.hanschen.org/2009/10/13/mouse-shortcuts-with-xbindkeys/).
