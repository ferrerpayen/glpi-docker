# Conteneur Docker-GLPI
## Pourquoi Docker ?
[Docker](https://www.docker.com/) est une plate-forme logicielle qui permet de concevoir, tester et déployer des applications rapidement. Docker intègre les logiciels dans des unités normalisées appelées conteneurs, qui rassemblent tous les éléments nécessaires à leur fonctionnement, dont les bibliothèques, les outils système, le code et l'environnement d'exécution. Ainsi, les applications pourront être facilement déployées, configurées et déplacées entre différents environnements.

## Procédure d'installation de Docker
Vous pouvez suivre la procédure d'installation suivante pour Debian, pour tout autre distribution/OS veuillez vous référer [ici](https://docs.docker.com/engine/install/).
### Configuration du repository
Il faut d'abord configurer le repository officiel Docker sur sa machine avant de pouvoir procéder à l'installation.
```
$ sudo apt-get update
$ sudo apt-get install ca-certificates curl gnupg lsb-release
$ sudo mkdir -m 0755 -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
### Installation de Docker
Il suffit simplement d'installer les paquets suivants.
```
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Pour vérifier que tout fonctionne correctement vous pouvez essayer de lancer le conteneur hello-world, celui-ci affiche simplement un message avant de s'arrêter.
```
$ sudo docker run hello-world
```
### Utiliser Docker sans sudo
Si vous le souhaitez, il est possible d'utiliser Docker sans sudo. Commencer par créer le groupe docker s'il n'existe pas déjà sur la machine.
```
$ sudo groupadd docker
```
Puis ajoutez votre utilisateur au groupe.
```
$ sudo usermod -aG docker $USER
```
Déconnecter puis reconnecter vous pour que les changements prennent effet. Il peut être nécessaire dans certains cas de redémarrer la machine. Vous pouvez essayer de lancer le conteneur hello-world sans sudo pour vérifier que cela fonctionne.
## Démarrage rapide
Commencez par cloner les sources du projet sur votre machine, par exemple avec la commande `git clone https://github.com/ferrerpayen/glpi-docker.git`.

Ensuite, éxécutez la commande `docker compose up -d --build` n'importe où dans l'arborescense du projet pour compiler les images et démmarer les conteneurs. 

Armez-vous d'un peu de patience, cette étape peut prendre une dizaine de minutes. Pas d'inquiétude, les prochains démarrages seront plutôt de l'ordre de la dizaine de secondes grâce au système de cache de Docker.

Avec `docker compose logs -f` vous pourrez observer l'avancée du démarrage une fois la compilation terminée. 

Une fois tout les conteneurs démarrés, rendez-vous à l'adresse de votre machine dans votre navigateur puis cliquez sur GLPI pour lancer son processus d'installation habituel.

Vous pouvez soit utiliser une base de données externe soit la base mariadb de la stack compose. Auquel cas, entrez les informations suivantes.

* Serveur SQL : `db.test.reslocal`
* Utilisateur SQL : `dbuser`
* Mot de passe SQL : `password`

Ensuite, sélectionnez la base `glpidata`.

Si vous utilisez une base de donnée externe, n'hésitez pas à supprimer le conteneur mariadb par la suite pour plus de performances.

Vous pouvez maintenant essayer GLPI sur Docker ! 

## Commandes
Voici les commandes que vous utiliserez sans doute le plus pour ce projet.
* `docker compose up|down` démarrer et arrêter les conteneurs.
    * `-d` éxécuter en arrière plan.
    * `--build` reconstruire les images.
* `docker compose logs` afficher les logs.
    * `-f` affiche les logs en continu. 
* `docker ps` affiche les conteneurs en cours d'exécution.
* `docker exec -it <id-conteneur> <commande-linux>` éxécute une commande dans le conteneur spécifié.
* `docker volume ls` lister les volumes.
* `docker volume rm <nom-volume>` supprimer un volume.

Vous pouvez en trouver d'autres [ici](https://docs.docker.com/engine/reference/commandline/cli/).

## Configuration du projet
### Fichiers de GLPI
Tous les fichiers de configuration de GLPI se trouvent dans `build/glpi`
* `Dockerfile` décrit à docker la manière de construire le conteneur GLPI.
* `downstream.php` permet de définir un dossier alternatif pour la config de GLPI dans le conteneur.
* `glpi.ini` fichier de configuration de PHP dans le conteneur GLPI.
* `local_define.php` permet de définir des dossier alternatifs pour les fichiers et logs de GLPI dans le conteneur.
* `vh.conf` défini un hôte virtuel apache pour GLPI dans le conteneur.
### Configuration des conteneurs
Les conteneurs sont configurés via le fichier `docker-compose.yml`. Le fichier définit 3 conteneurs en plus de celui de GLPI.
* `nginx` sert de reverse proxy pour donner l'accès aux autres services. Son fichier de configuration ainsi que la page d'accueil se situent dans le dossier `nginx` à la racine du projet.
* `mariadb` sert de base de données pour glpi. Par défaut, la base de données créée s'appelle `glpidata`, l'utilisateur s'appelle `dbuser` et son mot de passe ainsi que le mot de passe root sont `password`. On peut changer ces valeurs en modifiant les variables d'environement dans le fichier.
* `adminer` sert d'interface graphique pour la gestion des bases de données.

Ces trois conteneurs sont optionels, vous pouvez les supprimer si vous ne les utilisez pas. Vous pouvez aussi ajouter de nouveaux conteneurs et personnaliser ceux existants avec les paramètres suivants.

* `image:` nom de l'image et sa version `image:version` utilisée dans le conteneur. Vous pouvez trouver de nouvelles images sur [Docker Hub](https://hub.docker.com/).
* `build:` chemin vers un Dockerfile local pour utiliser sa propre image.
* `ports:` mapping entre un port local sur la machine et un port du conteneur `"port-local:port-conteneur"`.
* `networks: net: aliases:` nom du conteneur dans le réseau virtuel.
* `volumes:` mapping entre un volume virtuel ou un fichier local et un fichier du conteneur `fichier-local:fichier-conteneur`. Permet la persistance des données entre les redémarrages.
* `environment:` Permet d'initialiser des variables d'environement. Ces variables se trouvent généralement sur la page [Docker Hub](https://hub.docker.com/) de l'image.

Pour plus de détails et d'autres paramètres, veuillez vous référer [ici](https://docs.docker.com/compose/compose-file/).
### Persistance des données
Les données de MariaDB et de GLPI, ainsi que les fichiers d'installation, fichiers de configuration, fichiers de logs et fichiers du marketplace de ce dernier sont respectivement stockés dans les volumes Docker `mariadb`, `glpi-files`, `glpi-install`, `glpi-config`, `glpi-logs` et `glpi-marketplace`.

### Supprimer le fichier d'installation
Pour des raison de sécurité GLPI recommande de supprimer le fichier `install/install.php`. Vous pouvez suivre la procédure suivante pour régler le problème.
```
# Trouver l'id du conteneur GLPI
$ docker ps
# Suppression du fichier install/install.php
$ docker exec -it <id> rm ./install/install.php 
```
### Mettre à jour les conteneurs

Pour mettre à jouer les conteneurs il suffit simplement d'actualiser la valeur de leur version dans le champ `image` du fichier `docker-compose.yml`. Vous pouvez consulter les nouvelles versions disponibles sur [Docker Hub](https://hub.docker.com/) dans la section `Tags` d'une image.

Pour le conteneur GLPI c'est un peu différent. Celui-ci utilisant une image locale, il faut d'abord supprimer le volume `glpi-install`.
```
# Trouver le nom exact du volume
docker volume ls
# Supprimer le volume
docker volume rm XXX_glpi-install
```
Ensuite, rendez vous dans le dossier `/build/glpi/` et modifiez la version de GLPI dans le fichier `Dockerfile`.
```
# Changez la version de glpi ici
ENV VERSION=XX.XX.XX
```
> Attention après toute modification touchant aux images locales, il est nécessaire de les reconstruire afin que les changements soient pris en compte (option --build).
### Supprimer un conteneur

Pour supprimer un conteneur que vous n'utilisez pas, il suffit d'effacer les lignes qui correspondent à sa déclaration ainsi qu'a ses volumes dans le fichier `docker-compose.yaml`. Par exemple il faudrait effacer les lignes suivantes pour le conteneur adminer.
```
  adminer:
    image: adminer:4.8.1
    networks:
      net:
        aliases:
          - adminer.test.reslocal
```
Vous pouvez aussi commenter les lignes en ajoutant devant le charactère `#` pour le même effet.
### Ajouter des extensions PHP à GLPI

Toujours dans le fichier `/build/glpi/Dockerfile` on peut ajouter de nouvelles extensions PHP en modifiant la ligne suivante.
```
# Ajoutez des extensions ici
install-php-extensions XXX
```
Vous pouvez trouver la liste des extensions disponibles [ici](https://github.com/mlocati/docker-php-extension-installer#supported-php-extensions).
> Attention après toute modification touchant aux images locales, il est nécessaire de les reconstruire afin que les changements soient pris en compte (option --build).
