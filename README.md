# Auto-deploy-git

## Git push auto deploy light build
Ce fichier décrit comment construire un déploiement automatique "léger"

<!-- 
# ######################################################################################
# 
# Creation : 21/04/2020 -
# Version  : Insert n°1.0
# Fonctionalité du document :
#    Documentation de mise en oeuvre d'un déploiement automatique sur le serveur de développement. Ce déploiement sera mis en oeuvre lors d'une mise à jour (git push) de la 
#    master branch par le chef de projet <Nom du projet>.
#  
# ######################################################################################
# 
# List of Modifications / History :
#   21/04/2020 : Insert001 () :
#       - Configuration du déploiement automatique lors d'un git push.
#   
# List of Futur Feature (TODO) :
#   Déployer cette configuration sur un serveur de production.
#   Déployer cette configuration sur un repo distant de type Github.
#   
# ######################################################################################
-->

## Mise en place d'un déploiement automatique dans le cadre du déploiement continu des services web pour le projet <Nom du projet>

> **Point Important pour cette insertion :** il faut avoir une connaissance de GIT et avoir défini un flux de travail (workflow) au sein de l'équipe des développeurs. Seul le chef de projet doit pouvoir déployer la mise à jour de la branche master du projet <Nom du projet> ou un développeur tiers dans le cadre de l'application du workflow précédement évoqué.

### Prérequis :

Disposer d'un utilisateur (ex : utilisateur-a) ayant les droits d'écriture, lecture et exécution sur le groupe Node (gid=500) du serveur de développement (serveur node nomserveur.nn.oo).

Toute la procédure partira du principe que l'utilisateur utilisateur-a sera celui qui sera chargé du déploiement.

Cet utilisateur doit disposer des clés publiques root et node dans le fichier /home/node/utilisateur-a/.ssh/authorized_keys.
Ces dernières ne peuvent lui avoir été ajoutées que par l'utilisateur root.
De même que sa clé publique doit être présente sur le groupe Node dans le fichier /home/node/.ssh/authorized_keys.
A défaut, l'utilisateur root devra l'ajouter.

### Vérifier son appartenance au groupe Node ici en tant qu'utilisateur : 

Etre connecté en tant qu'utilisateur utilisateur-a.

```sh
[utilisateur-a@<nomServeur> ~]$ id
uid=1004(utilisateur-a) gid=1502(utilisateur-a) groups=1502(utilisateur-a),500(node) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

### Se connecter en tant qu'utilisateur node

```sh
[admin@<nomServeur> ~]$ ssh node@localhost
-bash-4.2$
```

## Procédure pour le Front

### Se placer sur le dossier du projet <nomProjet>

```sh
-bash-4.2$ cd /home/node/<nomProjet>
```

### Créer un dossier pour le déploiement du front

```sh
-bash-4.2$ mkdir <nomProjet>-app-deploy
```

### Créer un "dépot nu" (bare repositorie) dans un dossier de déploiement

```sh
-bash-4.2$ cd <nomProjet>-app-deploy
-bash-4.2$ git init --bare <nomProjet>-app.git
```

### Créer un fichier post-receive contenant un script hook dans chacun des "dépot nu" (bare repositories) front et back et le rendre exécutable

```sh
-bash-4.2$ cd <nomProjet>-app.git/hooks
-bash-4.2$ nano post-receive

#!/bin/bash
TARGET="/home/node/<nomProjet>/<nomProjet>-app"
GIT_DIR="/home/node/<nomProjet>/<nomProjet>-app-deploy/<nomProjet>-app.git"
BRANCH="master"

while read oldrev newrev ref
do
	# seulement la branche que l'on veut voir déployée
	if [ "$ref" = "refs/heads/$BRANCH" ];
	then
		echo "Ref $ref received. Deploying ${BRANCH} branch to production..."
		git --work-tree=$TARGET --git-dir=$GIT_DIR checkout -f $BRANCH
	else
		echo "Ref $ref received. Doing nothing: only the ${BRANCH} branch may be deployed on this server."
	fi
done

# TARGET est l'url du dossier de production du projet
# GIT_DIR est l'url du dossier contenant la logique de déploiement
# BRANCH est le nom de la branche que l'on souhaite déployer ici "master"

# On sauvegarde par Ctrl X 
# On valide par Y
# On conserve le nom de fichier post-receive

-bash-4.2$ chmod +x post-receive
```

### Ajouter une accroche de dépot (remote repositorie) depuis son dossier local de travail

> **Point Important pour cette partie :** si le développeur dispose de plusieurs ordinateurs et/ou d'environnements de développement pour le projet <nomProjet>-app (front) il faudra ajouter manuellement cette accroche depuis ces autres environnements. Cette ajout peut se faire à partir du terminal de son editeur de texte (IDE) de Git Bash ou d'un terminal natif d'OS.
Nous prendrons ici l'exemple de Git Bash.

Se placer dans le dossier local de développement du projet <nomProjet>-app 

```ssh
utilisateur-a@<id-UC> MINGW64 /c/Users/utilisateur-a/Documents/<nomProjet>-app (master)
$
```

### Vérification d'une accroche existante du fait d'inter-agir avec un dépot distant sur internet et hébergé sur Github.com

```ssh
utilisateur-a@<id-UC> MINGW64 /c/Users/utilisateur-a/Documents/<nomProjet>-app (master)
$git remote --verbose
origin  https://github.com/utilisateur-a/<nomProjet>-app.git (fetch)
origin  https://github.com/utilisateur-a/<nomProjet>-app.git (push)
```

### Ajout d'une accroche pour le dossier de déploiement et vérification de cet ajout

```ssh
utilisateur-a@<id-UC> MINGW64 /c/Users/utilisateur-a/Documents/<nomProjet>-app (master)
$git remote add production utilisateur-a@<nomServeur>:/home/node/<nomProjet>/<nomProjet>-app-deploy.git
$git remote --verbose
origin  https://github.com/utilisateur-a/<nomProjet>-app.git (fetch)
origin  https://github.com/utilisateur-a/<nomProjet>-app.git (push)
production      utilisateur-a@<nomServeur>:/home/node/<nomProjet>/<nomProjet>-app-deploy.git (fetch)
production      utilisateur-a@<nomServeur>:/home/node/<nomProjet>/<nomProjet>-app-deploy.git (push)
```

### Essais de push des modifications apportées pour une mise à jour de la production

> **Point Important pour cette partie :** Les modifications apportées au projet doivent avoir été commitées (cf: Workflow). Effectuer un git pull de la branche master, si cette dernière est à jour pusher.

```ssh
utilisateur-a@<id-UC> MINGW64 /c/Users/utilisateur-a/Documents/<nomProjet>-app (master)
$ git pull production master
utilisateur-a@<nomServeur>'s password:
From <nomServeur>:/home/node/<nomProjet>/<nomProjet>-app-deploy
 * branch            master     -> FETCH_HEAD
Already up to date.
utilisateur-a@<id-UC> MINGW64 /c/Users/utilisateur-a/Documents/<nomProjet>-app (master)
$ git push production master
utilisateur-a@<nomServeur>'s password:
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 322 bytes | 322.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Ref refs/heads/master received. Deploying master branch to production...
remote: Already on 'master'
To <nomServeur>:/home/node/<nomProjet>/<nomProjet>-app-deploy
   271e25c..0a9cc8a  master -> master
```

## Procédure pour le Back

### Se placer sur le dossier du projet <nomProjet>

```sh
-bash-4.2$ cd /home/node/<nomProjet>
```
### Créer un dossier pour le déploiement du back

```sh
-bash-4.2$ mkdir <nomProjet>-api-deploy
```

### Créer un "dépot nu" (bare repositorie) dans le dossier de déploiement

```sh
-bash-4.2$ cd <nomProjet>-api-deploy
-bash-4.2$ git init --bare <nomProjet>-api.git
```

### Créer un fichier post-receive contenant un script hook dans le "dépot nu" (bare repositorie) et le rendre exécutable

```sh
-bash-4.2$ cd <nomProjet>-api.git/hooks
-bash-4.2$ nano post-receive

#!/bin/bash
TARGET="/home/node/<nomProjet>/<nomProjet>-api"
GIT_DIR="/home/node/<nomProjet>/<nomProjet>-api-deploy/<nomProjet>-api.git"
BRANCH="master"

while read oldrev newrev ref
do
	# seulement la branche que l'on veut voir déployée
	if [ "$ref" = "refs/heads/$BRANCH" ];
	then
		echo "Ref $ref received. Deploying ${BRANCH} branch to production..."
		git --work-tree=$TARGET --git-dir=$GIT_DIR checkout -f $BRANCH
	else
		echo "Ref $ref received. Doing nothing: only the ${BRANCH} branch may be deployed on this server."
	fi
done

# TARGET est l'url du dossier de production du projet
# GIT_DIR est l'url du dossier contenant la logique de déploiement
# BRANCH est le nom de la branche que l'on souhaite déployer ici "master"

# On sauvegarde par Ctrl X 
# On valide par Y
# On conserve le nom de fichier post-receive

-bash-4.2$ chmod +x post-receive
```

### Ajouter une accroche de dépot (remote repositorie) depuis son dossier local de travail

> **Point Important pour cette partie :** si le développeur dispose de plusieurs ordinateurs et/ou d'environnements de développement pour le projet <nomProjet>-app (front) il faudra ajouter manuellement cette accroche depuis ces autres environnements. Cette ajout peut se faire à partir du terminal de son editeur de texte (IDE) de Git Bash ou d'un terminal natif d'OS.
Nous prendrons ici l'exemple de Git Bash.

Se placer dans le dossier local de développement du projet <nomProjet>-app 

```ssh
utilisateur-a@<id-UC> MINGW64 /c/Users/utilisateur-a/Documents/<nomProjet>-api (master)
$
```

### Vérification d'une accroche existante du fait d'inter-agir avec un dépot distant sur internet et hébergé sur Github.com

```ssh
utilisateur-a@<id-UC> MINGW64 /c/Users/utilisateur-a/Documents/<nomProjet>-api (master)
$git remote --verbose
origin  https://github.com/utilisateur-a/<nomProjet>-api.git (fetch)
origin  https://github.com/utilisateur-a/<nomProjet>-api.git (push)
```

### Ajout d'une accroche pour le dossier de déploiement et vérification de cet ajout

```ssh
utilisateur-a@<id-UC> MINGW64 /c/Users/utilisateur-a/Documents/<nomProjet>-api (master)
$git remote add production utilisateur-a@<nomServeur>:/home/node/<nomProjet>/<nomProjet>-api-deploy.git
$git remote --verbose
origin  https://github.com/utilisateur-a/<nomProjet>-api.git (fetch)
origin  https://github.com/utilisateur-a/<nomProjet>-api.git (push)
production      utilisateur-a@<nomServeur>:/home/node/<nomProjet>/<nomProjet>-api-deploy.git (fetch)
production      utilisateur-a@<nomServeur>:/home/node/<nomProjet>/<nomProjet>-api-deploy.git (push)
```

### Essais de push des modifications apportées pour une mise à jour de la production

> **Point Important pour cette partie :** Les modifications apportées au projet doivent avoir été commitées (cf: Workflow). Effectuer un git pull de la branche master, si cette dernière est à jour pusher.

```ssh
utilisateur-a@<id-UC> MINGW64 /c/Users/utilisateur-a/Documents/<nomProjet>-api (master)
$ git pull production master
utilisateur-a@<nomServeur>'s password:
From <nomServeur>:/home/node/<nomProjet>/<nomProjet>-api-deploy
 * branch            master     -> FETCH_HEAD
Already up to date.
utilisateur-a@<id-UC> MINGW64 /c/Users/utilisateur-a/Documents/<nomProjet>-api (master)
$ git push production master
utilisateur-a@<nomServeur>'s password:
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 322 bytes | 322.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Ref refs/heads/master received. Deploying master branch to production...
remote: Already on 'master'
To <nomServeur>:/home/node/<nomProjet>/<nomProjet>-api-deploy
   271e25c..0a9cc8a  master -> master
```

## Documentations des outils :

|Outils  | Documentation| 
|--|--|
| Git  | [https://git-scm.com/docs/git-remote](https://git-scm.com/docs/git-remote) |


## Source :
https://gist.github.com/thomasfr/9691385

### This file is a light extraction on the complete procedure who is in the source.
