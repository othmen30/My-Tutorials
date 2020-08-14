# Configuration initiale de serveur avec Ubuntu 20.04

    By Brian Boucheron
    PostedMay 13, 2020 2.4k views

###Introduction

Lorsque vous créez un nouveau serveur Ubuntu 20.04, vous devez effectuer quelques étapes de configuration importantes dans le cadre de la configuration de base. Ces étapes augmenteront la sécurité et la convivialité de votre serveur, et vous donneront une base solide pour les actions ultérieures.
Étape 1 - Connexion en tant que root

Pour vous connecter à votre serveur, vous devez connaître l’adresse IP publique de votre serveur. Vous aurez également besoin du mot de passe ou - si vous avez installé une clé SSH pour l'authentification - de la clé privée du compte de l'utilisateur root. Si vous n'êtes pas encore connecté à votre serveur, vous pouvez suivre notre guide sur Comment vous connecter à votre Droplet avec SSH, qui couvre ce processus en détail.

Si vous n'êtes pas encore connecté à votre serveur, connectez-vous maintenant en tant qu'utilisateur root en utilisant la commande suivante (remplacez la partie surlignée de la commande par l'adresse IP publique de votre serveur) :

    ssh root@your_server_ip

Acceptez l'avertissement concernant l'authenticité de l'hôte s'il apparaît. Si vous utilisez l'authentification par mot de passe, fournissez votre mot de passe root pour vous connecter. Si vous utilisez une clé SSH qui est protégée par une phrase de passe, il se peut que vous soyez invité à entrer la phrase de passe la première fois que vous utilisez la clé à chaque session. Si c'est la première fois que vous vous connectez au serveur avec un mot de passe, vous pouvez également être invité à changer le mot de passe root.
À propos de Root

L'utilisateur root est l'utilisateur administratif dans un environnement Linux qui dispose de privilèges très larges. En raison des privilèges accrus du compte root, il est déconseillé de l'utiliser régulièrement. En effet, une partie du pouvoir inhérent au compte root est sa capacité à effectuer des changements très destructeurs, même par accident.

L'étape suivante consiste à mettre en place un nouveau compte d'utilisateur avec des privilèges réduits pour une utilisation quotidienne. Plus tard, nous vous apprendrons comment obtenir des privilèges accrus uniquement lorsque vous en avez besoin.
Étape 2 - Création d'un nouvel utilisateur

Une fois que vous êtes connecté en tant que root, nous sommes prêts à ajouter le nouveau compte d'utilisateur. Dans le futur, nous nous connecterons à ce nouveau compte au lieu de root.

Cet exemple crée un nouvel utilisateur appelé sammy, mais vous devez le remplacer par le nom d'utilisateur qui vous convient :

    adduser sammy

Quelques questions vous seront posées, en commençant par le mot de passe du compte.

Saisissez un mot de passe fort et, éventuellement, remplissez les informations complémentaires si vous le souhaitez. Ce n'est pas obligatoire et vous pouvez juste appuyer sur ENTER dans n'importe quel champ que vous souhaitez ignorer.
Étape 3 - Attribution des privilèges administratifs

Maintenant, nous avons un nouveau compte utilisateur avec des privilèges de compte ordinaires. Cependant, nous devons parfois effectuer des tâches administratives.

Pour éviter d'avoir à nous déconnecter de notre utilisateur normal et à nous reconnecter en tant que compte root, nous pouvons configurer ce que l'on appelle des privilèges de super-utilisateur ou de root pour notre compte ordinaire. Cela permettra à notre utilisateur normal d'exécuter des commandes avec des privilèges administratifs en plaçant le mot sudo avant chaque commande.

Pour ajouter ces privilèges à notre nouvel utilisateur, nous devons ajouter l'utilisateur au groupe sudo. Par défaut, sur Ubuntu 20.04, les utilisateurs qui sont membres du groupe sudo sont autorisés à utiliser la commande sudo.

En tant que root, exécutez cette commande pour ajouter votre nouvel utilisateur au groupe sudo (remplacez le nom d'utilisateur en surbrillance par celui de votre nouvel utilisateur) :

    usermod -aG sudo sammy

Maintenant, lorsque vous êtes connecté en tant qu'utilisateur régulier, vous pouvez taper sudo avant les commandes pour effectuer des actions avec des privilèges de super-utilisateur.
Étape 4 - Mise en place un pare-feu de base

Les serveurs Ubuntu 20.04 peuvent utiliser le pare-feu UFW pour s'assurer que seules les connexions à certains services sont autorisées. Nous pouvons très facilement mettre place un pare-feu de base en utilisant cette application.

Remarque : si vos serveurs tournent sur DigitalOcean, vous pouvez éventuellement utiliser les pare-feux Cloud de DigitalOcean au lieu du pare-feu UFW. Nous recommandons de n'utiliser qu'un seul pare-feu à la fois pour éviter les règles contradictoires qui peuvent être difficiles à déboguer.

Les applications peuvent enregistrer leurs profils dans UFW lors de leur installation. Ces profils permettent à UFW de gérer ces applications par nom. OpenSSH, le service qui nous permet maintenant de nous connecter à notre serveur, dispose d'un profil enregistré avec UFW.

Vous pouvez le voir en tapant :

    ufw app list

Output
Available applications:
  OpenSSH

Nous devons nous assurer que le pare-feu autorise les connexions SSH afin de pouvoir nous reconnecter la prochaine fois. Nous pouvons autoriser ces connexions en tapant :

    ufw allow OpenSSH

Après quoi, nous pouvons activer le pare-feu en tapant :

    ufw enable

Tapez y et appuyez sur ENTER pour continuer. Vous pouvez voir que les connexions SSH sont toujours autorisées en tapant :

    ufw status

Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)

Comme le pare-feu bloque actuellement toutes les connexions, sauf les connexions SSH, si vous installez et configurez des services supplémentaires, vous devrez ajuster les paramètres de pare-feu pour autoriser le trafic. Vous pouvez apprendre quelques opérations courantes UFW dans notre guide des fondamentaux UFW.
Étape 5 - Activation de l'accès externe pour votre utilisateur ordinaire

Maintenant que nous avons un utilisateur régulier pour un usage quotidien, nous devons nous assurer que nous pouvons SSH directement dans le compte.

Remarque : tant que vous n'avez pas vérifié que vous pouviez vous connecter et utiliser sudo avec votre nouvel utilisateur, nous vous recommandons de rester connecté en tant que root. De cette façon, si vous avez des problèmes, vous pouvez les résoudre et apporter les changements nécessaires en tant que root. Si vous utilisez une Droplet DigitalOcean et que vous rencontrez des problèmes avec votre connexion SSH root, vous pouvez vous connecter à la Droplet en utilisant la Console DigitalOcean.

Le processus de configuration de l'accès SSH pour votre nouvel utilisateur dépend du fait que le compte root de votre serveur utilise un mot de passe ou des clés SSH pour l'authentification.
Si le compte root utilise l'authentification par mot de passe

Si vous vous êtes connecté à votre compte root à l'aide d'un mot de passe, l'authentification par mot de passe est alors activée pour SSH. Vous pouvez accéder à votre nouveau compte utilisateur en ouvrant une nouvelle session de terminal et en utilisant SSH avec votre nouveau nom d'utilisateur :

    ssh sammy@your_server_ip

Après avoir saisi votre mot de passe d'utilisateur habituel, vous serez connecté. N'oubliez pas que si vous devez gérer une commande avec des privilèges administratifs, tapez sudo devant comme ceci :

    sudo command_to_run

Votre mot de passe d'utilisateur habituel vous sera demandé lors de la première utilisation de sudo à chaque session (et périodiquement par la suite).

Pour renforcer la sécurité de votre serveur, nous vous recommandons vivement de configurer des clés SSH plutôt que d'utiliser une authentification par mot de passe. Suivez notre guide sur la Configuration des clés SSH sur Ubuntu 20.04 pour apprendre comment configurer l'authentification par clé.
Si le compte root utilise l'authentification par clé SSH

Si vous vous êtes connecté à votre compte root en utilisant des clés SSH, l'authentification par mot de passe est désactivée pour SSH. Vous devrez ajouter une copie de votre clé publique locale au fichier ~/.ssh/authorized_keys du nouvel utilisateur pour vous connecter avec succès.

Comme votre clé publique se trouve déjà dans le fichier ~/.ssh/authorized_keys du compte root sur le serveur, nous pouvons copier ce fichier et la structure des répertoires sur notre nouveau compte utilisateur dans notre session existante.

La commande rsync représente la façon la plus simple de copier les fichiers avec la propriété et les autorisations correctes. Cela permet de copier le répertoire .ssh de l'utilisateur root, de préserver les permissions et de modifier les propriétaires de fichiers, le tout en une seule commande. Veillez à modifier les parties surlignées de la commande ci-dessous pour qu'elles correspondent à votre nom d'utilisateur habituel :

Remarque : la commande rsync traite différemment les sources et les destinations qui se terminent par une barre oblique et celles sans barre oblique. Lorsque vous utilisez rsync ci-dessous, assurez-vous que le répertoire source (~/.ssh) ne comporte pas de barre oblique (vérifiez que vous n'utilisez pas ~/.ssh/).

Si vous ajoutez accidentellement une barre oblique à la fin de la commande, rsync copiera le contenu du répertoire ~/.ssh du compte root dans le répertoire personnel de l'utilisateur sudo au lieu de copier toute la structure du répertoire ~/.ssh. Les fichiers se trouveront au mauvais endroit et les SSH ne seront pas en mesure de les trouver et de les utiliser.

    rsync --archive --chown=sammy:sammy ~/.ssh /home/sammy

Maintenant, ouvrez une nouvelle session de terminal sur vous machine locale, et utilisez le SSH avec votre nouveau nom d'utilisateur :

    ssh sammy@your_server_ip

Vous devez être connecté au nouveau compte utilisateur sans utiliser de mot de passe. N'oubliez pas que si vous devez gérer une commande avec des privilèges administratifs, tapez sudo devant comme ceci :

    sudo command_to_run

Votre mot de passe d'utilisateur habituel vous sera demandé lors de la première utilisation de sudo à chaque session (et périodiquement par la suite).
Que faire maintenant ?

À ce stade, vous disposez d'une base solide pour votre serveur. Vous pouvez désormais installer tous les logiciels dont vous avez besoin sur votre serveur.
Qu'avez-vous pensé de cette traduction?
Was this helpful?
0
Report an issue
