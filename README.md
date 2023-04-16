# projet de liste d'étudiants

Veuillez trouver les spécifications en cliquant [ici](https://github.com/diranetafen/student-list.git "ici")

!["Crédit image : eazytraining.fr"](https://eazytraining.fr/wp-content/uploads/2020/04/pozos-logo.png) ![projet](https://user-images.githubusercontent .com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)

------------

Prénom : Nesta

Nom : Kengni

Pour le 11ème Bootcamp DevOps d'Eazytraining

Période : janvier-Fevrier-mars
dimanche 16 Avril 2023

## Application

J'ai dû déployer une application nommée "*student_list*", qui est très basique et permet à POZOS d'afficher la liste de certains étudiants avec leur âge.

L'application student_list comporte deux modules :

- Le premier module est une API REST (avec authentification de base nécessaire) qui envoie la liste des souhaits de l'étudiant sur la base d'un fichier JSON
- Le deuxième module est une application Web écrite en HTML + PHP qui permet à l'utilisateur final d'obtenir une liste d'étudiants

## Le besoin

Mon travail consistait à :
1) construire un conteneur pour chaque module
2) les faire interagir les uns avec les autres
3) fournir un registre privé

## Mon plan

Tout d'abord, laissez-moi vous présenter les cinq***fichiers*** de ce projet et leur rôle

Ensuite, je vous montrerai comment j'ai ***construit*** et testé l'architecture pour justifier mes choix

La troisième et dernière partie sera sur le point de fournir le processus de *** déploiement *** que je suggère pour cette application.


### Le rôle des fichiers

Dans ma livraison vous pouvez trouver trois fichiers principaux : un ***Dockerfile***, un ***docker-compose.yml*** et un ***docker-compose.registry.yml***

- docker-compose.yml : pour lancer l'application (API et web app)
- simple_api/student_age.py : contient le code source de l'API en python
- simple_api/Dockerfile : pour construire l'image de l'API avec le code source dedans
- simple_api/student_age.json : contient le nom de l'étudiant avec l'âge au format JSON
- index.php : page PHP où l'utilisateur final sera connecté pour interagir avec le service afin de répertorier les étudiants avec leur âge.


# Construire et tester

Considérant que vous venez de cloner ce référentiel, vous devez suivre ces étapes pour préparer l'application 'student_list' :
 ## Build et test
1) Changez de répertoire et création de l'image du conteneur d'api :

```bash
cd ./student-list/simple_api
docker build . -t devopsteams:v1
docker images 
```
> ![1-docker images](images/image%20dans%20mon%20registre%20priv%C3%A9.JPG)

2) Création du contenainer et Exécution de la commande curl pour etre sur que l'api répond correctement :

```bash
docker run -- name api -d -p 5000:5000 devopsteams:v1
 curl -u toto:python -X GET http://192.168.56.5:5000/pozos/api/v1.0/get_student_ages
```
> ![Resultat](https://user-images.githubusercontent.com/101605739/224588523-a842cd26-c5d5-4338-8547-2e31578655c9.jpg)

3) Revenez au répertoire racine du projet et exécutez le conteneur d'api backend avec ces arguments :

```bash
cd ..
docker run --rm -d --name=api.student_list --network=student_list.network -v ./simple_api/:/data/ api.student_list.img
docker ps
```
> ![3-docker ps](https://user-images.githubusercontent.com/101605739/224589378-abcc3f7d-d5c6-4a81-ba28-767cb6cd7b7c.jpg)

Comme vous pouvez le voir, le conteneur principal de l'api écoute le port 5000.
Ce port interne peut être atteint par un autre conteneur du même réseau, j'ai donc choisi de ne pas l'exposer.

J'ai également dû monter le répertoire local `./simple_api/` dans le répertoire de conteneur interne `/data/` afin que l'API puisse utiliser la liste `student_age.json`


> ![4-./simple_api/:/data/](https://user-images.githubusercontent.com/101605739/224589839-7a5d47e6-fdff-40e4-a803-99ebc9d70b03.png)


4) Mettez à jour le fichier `index.php` :

Vous devez mettre à jour la ligne suivante avant d'exécuter le conteneur de site Web pour que ***api_ip_or_name*** et ***port*** correspondent à votre déploiement
    ` $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';`

Grâce aux fonctions dns de notre réseau de type pont, nous pouvons facilement utiliser le nom du conteneur api avec le port que nous avons vu juste avant pour adapter notre site

```bash
sed -i s\<api_ip_or_name:port>\api.student_list:5000\g ./website/index.php
```
> ![5-api.student_list:5000](https://user-images.githubusercontent.com/101605739/224590958-49c2ce64-c9a0-4655-93da-552f27f78b2f.png)

5) Exécutez le conteneur de l'application Web frontale :

Le nom d'utilisateur et le mot de passe sont fournis dans le code source `.simple_api/student_age.py`

> ![6-id/passwd](https://user-images.githubusercontent.com/101605739/224590363-0fdd56ae-9fb9-45e7-8912-64a6789faa9e.png)

```bash
docker run --rm -d --name=webapp.student_list -p 80:80 --network=student_list.network -v ./website/:/var/www/html -e USERNAME=toto -e PASSWORD=python php : apache
docker ps
```
> ![7-docker ps](https://user-images.githubusercontent.com/101605739/224591443-344fd2cd-ddbc-4780-bbc5-7cc0bdac156f.jpg)


6) Testez l'api via le frontend :

6a) En ligne de commande :

La commande suivante demandera au conteneur frontal de demander l'API backend et