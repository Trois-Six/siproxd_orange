siproxd-orange
==============

**L'auteur de ce projet ne travaille plus dessus suite à des pressions d'Orange.** 
Voir [ce message sur son blog](http://x0r.fr/blog/47) pour plus d'information.

**Lisez impérativement ce document.**  Il contient des instructions importantes 
afin de faire fonctionner ce plugin.

**Vous devrez impérativement utiliser une version ou snapshot postérieure au
26 mai 2014 de siproxd pour que ce plugin fonctionne correctement dans les
deux sens**.  Dans le cas contraire, **vous ne pourrez pas recevoir d'appels
via SIP**.

Ce plugin est à utiliser à vos propres risques et périls.  Je dégage toute 
responsabilité face aux conséquences de l'utilisation du proxy avec ce plugin.

Description
-----------

Ceci est un projet de plugin pour siproxd afin de permettre à un client SIP de 
s'authentifier et de passer des appels via le [service Livephone d'Orange](http://assistance.orange.fr/l-application-livephone-sur-pc-4898.php).

Les serveurs SIP du Livephone utilisent une version modifiée de la RFC 3261, ce
qui rend de fait impossible la compatibilité de leur service avec autre chose
que les applications propriétaires (pour Windows, iOS et Android) qui vont
avec.  Ce plugin se charge donc de réécrire certains messages SIP afin
d'assurer l'**interopérabilité** entre n'importe quel client SIP et
l'infrastructure d'Orange.

Le plugin permet le REGISTER auprès du proxy sortant utilisé par le Livephone,
et ainsi possible de passer des appels (INVITE) vers l'extérieur et recevoir
des appels depuis un autre téléphone.  Exactement de la même manière que si
vous aviez branché un téléphone analogique sur la prise prévue à cet effet sur
la Livebox, mais via SIP.


Pour les détails techniques de l'implémentation, vous pouvez lire l'article à 
l'adresse http://x0r.fr/blog/36.


Compilation
-----------

Avant de commencer, il vous faudra installer :
 
 * pkg-config (ou pkgconf sous FreeBSD) ;
 * libxml2 (-dev) version 2.9.1 ou supérieure ;
 * libcurl (-dev) avec prise en charge OpenSSL ;
 * libosip2 (-dev) version 3.6.0 ou supérieure ;
 * libltdl (-dev) fournie avec libtool ;
 * siproxd : deux choix possibles :

   * siproxd (0.8.2 ou supérieure, lorsque cette version existera), si
     possible, ou [cette snapshot patchée](http://x0r.fr/blogstuff/siproxd-15Sep2014-patched.tar.gz), ou
   * un exemplaire de l'archive des sources de siproxd (snapshot du 26 mai 2014
     ou postérieure) afin de compiler ce plugin (les fichiers .h nécessaires ne
     sont généralement pas distribués avec le binaire).

Les versions 0.1.3 et antérieures de siproxd-orange ne doivent plus être 
utilisées, car celles-ci ont une mauvaise implémentation de l'algorithme 
d'authentification.

Les versions 0.1.2 et antérieures requièrent la version 7.36.0 ou supérieure de
libcurl.  Les versions 0.1.3 et supérieures peuvent fonctionner avec des
versions plus anciennes de libcurl.  La version fournie par Debian Wheezy 
suffit, par exemple.

### Compiler siproxd

Pour que le plugin fonctionne, il vous faudra compiler [une snapshot récente](http://siproxd.sourceforge.net/index.php?op=snapshot) (postérieure au 26 mai 2014) ou installer la version 0.8.2 de siproxd
lorsque cette version existera.

Étant donné que ces snapshots de siproxd ont besoin de patchs supplémentaires
sur certains systèmes d'exploitation, je vous propose également [ma version
patchée de siproxd](http://x0r.fr/blogstuff/siproxd-15Sep2014-patched.tar.gz) qui devrait également fonctionner.

Dans le cas d'une snapshot, compilez et installez les sources en vous référant
aux instructions fournies.

### Compiler le plugin

Décompressez l'archive des sources (X.Y.Z représentant la version du plugin) :

	% tar xvzf siproxd_orange-X.Y.Z.tar.gz
	% cd siproxd_orange-X.Y.Z

Décompressez l'archive des sources de siproxd dans le répertoire actuel :

	% tar xvzf /chemin/vers/siproxd-0.8.2dev.tar.gz

Créez un lien symbolique vers les sources de siproxd comme ceci :

	% ln -s siproxd-0.8.2dev siproxd

Si vous avez récupéré les sources depuis le dépôt Mercurial, exécutez la
commande :

	% autoreconf -i

Exécutez les commandes suivantes :

	% ./configure
	% make

Exécutez `make install` en tant que root, ou via sudo.


Si tout se passe bien, plusieurs fichiers `.a`, `.la` et `.so` auront été 
installés dans `/usr/lib/siproxd` ou `/usr/local/lib/siproxd`.  Si ce n'est pas 
le cas, ajoutez une option `--prefix=/chemin/vers/prefixe` à `./configure`.  Le 
plugin sera alors installé dans le répertoire 
`/chemin/vers/prefixe/lib/siproxd/`.



Configuration du proxy
----------------------

Afin que le plugin fonctionne correctement, vous devrez ajouter les directives 
suivantes dans votre `siproxd.conf` (dans `/etc` ou `/usr/local/etc`, selon votre
installation) :

	load_plugin = plugin_orange.la
	plugin_orange_username = nom.prenom@orange.fr
	plugin_orange_password = mon_password_mail

Vérifiez aussi que le `plugindir` soit correct et pointe bien vers le répertoire où sont
installés les plugins (y compris celui-ci).


Configuration des clients
-------------------------

Consultez la [matrice de compatibilité](https://bitbucket.org/xtab/siproxd_orange/wiki/Home) dans le [wiki](https://bitbucket.org/xtab/siproxd_orange/wiki/Home) du projet.
Celui-ci regroupe toutes les instructions pour les clients SIP testés avec ce 
logiciel.

Diagnostic
----------

La procédure recommandée pour obtenir des informations de diagnostic est la suivante :

 1. Positionner `daemonize = 0` dans le fichier de configuration siproxd.conf ;

 2. Exécuter siproxd à la main à l'aide de la commande suivante :

		/sbin/siproxd -d 4132 2>&1 | tee siproxd.log

    Si `/sbin/siproxd` ne fonctionne pas, essayez avec
    `/usr/local/sbin/siproxd`  ;

 3. Récupérer le fichier siproxd.log ;

 4. Optionnel mais recommandé : cacher les numéros de téléphone, les
    identifiants et les adresses mail dans le fichier de traces ;

 5. Remettre `daemonize = 1` dans siproxd.conf avant de remettre en production.

Limitations
-----------

 * Ce plugin est sujet aux mêmes limitations techniques que l'application
   Livephone d'origine.  Par exemple, `siproxd` **doit** tourner sur une
   machine reliée à votre connexion Orange.

 * Je n'ai absolument pas testé le comportement de ce plugin avec d'autres
   fonctionnalités de siproxd.  En particulier, ce plugin risque de modifier des
   messages SIP qui ne sont pas forcément destinés à Orange.

 * La documentation fournie par Orange semble suggérer qu'enregistrer plusieurs
   clients SIP simultanément chez eux est une mauvaise idée.  Aussi, je vous
   déconseille de faire la même chose avec ce plugin de proxy.  De toute façon,
   pour partager une ligne SIP avec plusieurs téléphones, le mieux reste
   d'utiliser Asterisk.

 * Il arrive parfois qu'en appelant une personne avec Linphone, il soit
   impossible de l'entendre.  Raccrocher et retenter l'appel résoud
   généralement le problème.  Si vous n'entendez pas la tonalité après avoir
   composé un numéro, c'est généralement mauvais signe.  Je n'ai pas rencontré
   ce problème avec d'autres clients SIP pour le moment.
