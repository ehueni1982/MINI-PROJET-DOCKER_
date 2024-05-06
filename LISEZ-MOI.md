# Projet student-list

Veuillez trouver la consigne en cliquant [ici](https://github.com/diranetafen/student-list.git "ici")

!["Crédit image : eazytraining.fr"](https://eazytraining.fr/wp-content/uploads/2020/04/pozos-logo.png) ![projet](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)

------------
Prénom : Ehueni Barthelémy

Nom : ANGORA

 DevOps Bootcamp d’Eazytraining

Période : mars-avril-mai

Dimanche 5 mai 2024
////////////////


Application
J’ai dû déployer une application nommée « student_list », qui est très basique et permet à POZOS d’afficher la liste de certains élèves avec leur âge.

student_list’application comporte deux modules :

Le premier module est une API REST (avec authentification de base nécessaire) qui envoie la liste de souhaits de l’étudiant sur la base d’un fichier JSON
Le deuxième module est une application web écrite en HTML + PHP qui permet à l’utilisateur final d’obtenir une liste d’étudiants
Le besoin

Mon travail consistait à :

Construire un conteneur pour chaque module
les faire interagir les uns avec les autres sur le même réseau
Fournir un registre privé sur le même réseau
Mon plan
Tout d’abord, je vous présente les six fichiers de ce projet et leur rôle

Ensuite, je vous montrerai comment j’ai construit et testé l’architecture pour justifier mes choix

La troisième et dernière partie sera sur le point de fournir le processus de déploiement que je suggère pour cette application.

Le rôle des fichiers
Dans ma livraison, vous pouvez trouver trois fichiers principaux : un Dockerfile, un docker-compose.yml et un docker-compose.registry.yml

docker-compose.yml : pour lancer l’application (API et web app)
docker-compose.registry.yml : lancer le registre local et son frontend
simple_api/student_age.py : contient le code source de l’API en python
simple_api/Dockerfile : pour construire l’image de l’API avec le code source à l’intérieur
simple_api/student_age.json : contient le nom de l’élève avec l’âge au format JSON
index.php : page PHP où l’utilisateur final sera connecté pour interagir avec le service afin de répertorier les étudiants avec leur âge.
Construire et tester
Étant donné que vous venez de cloner ce dépôt, vous devez suivre les étapes suivantes pour préparer l’application 'student_list' :

1) Changez de répertoire et construisez l’image conteneur de l’api :
cd ./mini-projet-docker/simple_api
docker build . -t api-pozos:1
docker images
> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/fab442f5-8f66-4c87-ba9a-388914ae2199)

2) Créez un réseau de type pont pour que les deux conteneurs puissent se contacter par leurs noms grâce aux fonctions dns :
docker network create pozos
docker network ls

Revenez au répertoire racine du projet et exécutez le conteneur de l’api backend avec les arguments suivants :
cd ..
docker run --rm -d --network pozos --name test-api-pozos -v ${PWD}/student_age.json:/data/student_age.json -p 4000:5000 api-pozos:1
docker ps
> ![4-./simple_api/:/data/](https://user-images.githubusercontent.com/101605739/224589839-7a5d47e6-fdff-40e4-a803-99ebc9d70b03.png)

Comme vous pouvez le voir, le conteneur backend de l’API est à l’écoute du port 4000. Ce port interne est accessible par un autre conteneur du même réseau, j’ai donc choisi d’exposer sur le port 5000.

J’ai également dû monter le répertoire local dans le répertoire interne du conteneur pour que l’API puisse utiliser la liste${PWD}/:/data/student_age.json

./simple_api/ :/data/

4) Mettre à jour le fichier :index.php
Vous devez mettre à jour la ligne suivante avant d’exécuter le conteneur de site web pour adapter api_ip_or_name et le port à votre déploiement  $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';

Grâce aux fonctions dns de notre réseau de type bridge, nous pouvons facilement utiliser le nom du conteneur api avec le port que nous avons vu juste avant pour adapter notre site web

> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/1e5c98c9-1de4-4a93-9b54-fe36692b20d1)

Testez l’api via le frontend :
5a) En ligne de commande :

La commande suivante demandera au conteneur frontal de demander l’API backend et de vous montrer la sortie. L’objectif est de tester à la fois si l’api fonctionne et si le frontend peut obtenir la liste des étudiants à partir de celle-ci.

curl -u toto:python -X GET http://127.0.0.1:4000/pozos/api/v1.0/get_student_ages
> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/1c7471cb-edd5-4188-b6aa-94140f9e857c)

5b) Utilisation d’un navigateur web :IP:4000

Si vous exécutez l’application sur un serveur distant ou une machine virtuelle (par exemple, provisionnée par le fichier vagrant d’eazytraining), veuillez trouver votre adresse IP en tapant sur centoshostname -I
hostname -I
``
![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/8d7e1bf1-ec0b-4cf5-8b92-9bea54491624)

- Si vous travaillez sur PlayWithDocker, ouvrez simplement le port 8082 ` sur l'interface graphique
- Sinon, saisir `localhost:8082`

## Déploiement

Une fois les tests réussis, nous pouvons désormais « composer » notre infrastructure en plaçant les paramètres « docker run » au format ***infrastructure as code*** dans un fichier `docker-compose.yml` file.

1) Exécutez l'application (api + webapp) :

Comme nous avons déjà créé l'image de l'application, il ne vous reste plus qu'à exécuter :

```bash
docker-compose up -d
![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/a4723207-87f5-422d-8518-8eea86eb9839)

Docker-compose permet de choisir quel conteneur doit démarrer en premier. Le conteneur d’api sera le premier car j’ai spécifié que l’application web.depends_on:

> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/df3e9312-7cc2-4a28-878c-e169fb2d2bb7)
> 
![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/0e4e15cc-50aa-4199-a940-89fed442a682)

Et l’application fonctionne : ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/241aa70a-0bb9-42e5-a4ca-0c0ea22e6008)

2) Créer un registre et son frontend
J’ai utilisé image pour le registre, et pour son interface graphique frontale et passé quelques variables d’environnement :registry:2joxit/docker-registry-ui:static

>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/264a171e-825a-428c-ac83-0400157ffb28)

Par exemple, nous serons en mesure de supprimer des images du registre via l’interface graphique.

docker-compose -f docker-compose.registry.yml up -d
3) Exécutez une image sur le registre et testez l’interface graphique
Vous devez le renommer avant (c’est facultatif) ::latest

NB : pour cet exercice, j’ai laissé les identifiants dans le fichier .yml.

```bash
docker run -d -p 5000:5000 --name registry-pozos --network student-list_api-pozos registry:2
```
>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/ea0feac3-dcd2-4a48-ae73-6f37a537ab32)

4) Renommage le registre et Push

>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/ec34e0aa-ff95-4e26-8810-a8fbe5fd0b08)

  
>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/d733e7db-b056-495c-bff5-e29f17d48bf1)


> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/715bce28-2e53-49a8-a39d-2116fcb763f6)
> 


> ![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/180e0814-f033-4120-adda-3fe1209a1b2c)

5) Push d'une autre image de registre

Images de Docker
''''bash

docker image tag joxit/docker-registry-ui :1.5-static localhost :5000/joxit/docker-registry-ui :1.5-angora
'''''''
>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/f009a6a3-314f-4275-8e61-e505197818bf)

>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/e3aa2128-a23f-430b-b64b-23e1bc22fe17)

Docker push localhost:5000/joxit/docker-registry-ui:1.5-angora
>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/db147d9a-e431-4332-8439-f6bfd1d52133)

>![image](https://github.com/ehueni1982/MINI-PROJET-DOCKER_/assets/157939806/f7ecd30d-1b17-421e-a646-60757faab9ec)
''''''''''''''''''''''''''''''''''''

Ceci conclut mon rapport d’exécution de mini-projet Docker.
Tout au long de ce projet, j’ai eu l’occasion de créer une image Docker personnalisée, de configurer des réseaux et des volumes, et de déployer des applications à l’aide de docker-compose. Dans l’ensemble, ce projet a été une expérience enrichissante qui m’a permis de renforcer mes compétences techniques et de mieux comprendre les principes des microservices. Je suis maintenant mieux outillé pour m’attaquer à des projets similaires à l’avenir et contribuer à l’amélioration des processus de conteneurisation et de déploiement au sein de mon équipe et de mon organisation.

![octocat](https://myoctocat.com/assets/images/base-octocat.svg) 
