
<div align="center">

![Docker Build](https://img.shields.io/badge/docker-build-green)       ![Docker Build](https://img.shields.io/badge/dockercompose-up-green)            

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=Jenkins&logoColor=white)         ![Docker](https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white)                ![Docker](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)         

</div>
<div align="center">
Deploy a Jenkins server with Docker containers<br/>
+ Create a CI/CD pipeline

![project](https://github.com/Tony-Dja/Jenkins_CI-CD_in_Docker/blob/70fafb0b03686fa3c5a4bc7630d4aedd708d3ca9/screenshots/jenkins-docker.png)
</div>

# Dockerfile

To deploy the Jenkins server, we build an image from => Jenkins2.375<br/>
Just install inside, the Docker CLI, Docker compose and BlueOcean plugin from Jenkins to show graphical interface for pipelines.

We build the Dockerfile directly from Docker-compose.yml, but you can also build with GitHub actions, Push image on DockerHub and Pull it from the Docker compose file.<br/>
You could find this workflow with GitHub Actions in the ".github" folder.

```
FROM jenkins/jenkins:2.375-jdk11

USER root

RUN apt-get update && apt-get install -y apt-transport-https \
       ca-certificates curl gnupg2 \
       software-properties-common

RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -

RUN apt-key fingerprint 0EBFCD88

RUN add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"

RUN apt-get update && apt-get install -y docker-ce-cli

RUN curl -L "https://github.com/docker/compose/releases/download/`curl -fsSLI -o /dev/null -w %{url_effective} https://github.com/docker/compose/releases/latest | sed 's#.*tag/##g' && echo`/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \ 
    chmod +x /usr/local/bin/docker-compose && \ 
    ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

USER jenkins

RUN jenkins-plugin-cli --plugins blueocean
```


# Docker compose

To run containers just execute the following command :

```
docker compose up -d
```

The jenkins service is exposed on port 8080

```
services:
  jenkins:
    container_name: jenkins-server
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - 8080:8080
      - 50000:50000
    environment:
      - DOCKER_HOST=tcp://docker:2376
      - DOCKER_CERT_PATH=/certs/client
      - DOCKER_TLS_VERIFY=1
    networks:
      - local_jenkins
    hostname: jenkins
    restart: always
    volumes:
      - jenkins-data:/var/jenkins_home
      - jenkins-docker-certs:/certs/client:ro
    depends_on:
      - docker
```

Check the containers running :

```
docker ps
```
![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/679cf211c27019b0729d866204757bb8b4718964/screenshots/docker-ps.png)


Now to display the Jenkins server on your browser :

```
http://localhost:8080
```

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/679cf211c27019b0729d866204757bb8b4718964/screenshots/first.png)


For the first start you have to retrieve the initial password generate by Jenkins.<br/>
To do that you must connecting on the jenkins-server container and display the password file.

```
docker exec -it jenkins-server /bin/bash
```

in the container :

```
cat /var/jenkins_home/secrets/initialAdminPassword 
```

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/679cf211c27019b0729d866204757bb8b4718964/screenshots/first.png)



- docker-compose.yml: déploiement de l'application (API et frontend)
- Dockerfile: Conteneurisation de l'API Rest
- student_age.json: contient la liste des étudiants avec leur nom et age au format JSON
- student_age.py: code source python de l'API
- index.php: Page affichant le frontend et envoie la requête au backend


# Builder l'image

Exécutez le fichier Dockerfile /simple_api/Dockerfile pour conteneuriser l'API

Image Base => python:2.7-buster <br/>
Copie du code source => COPY . /<br/>
Création d'un volume afin de monter les données JSON => VOLUME /data <br/>
Exposition du conteneur => port 5000


<strong>- Builder</strong>
```
Docker build -t api-service .
```

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/e12bb5924d71eef708ffdec59bfae04766c943ea/screenshots/build-api-service.png)


<strong>- Vérifier les images</strong>
```
docker images
```

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/e12bb5924d71eef708ffdec59bfae04766c943ea/screenshots/docker-images.png)


# Lancer le conteneur

Les données JSON sont chargées au lancement en spécifiant un volume BindMount -v pointant sur le fichier "student_age.json"


<strong>- Lancement</strong>
```
docker run -d --name container-api-service -p 5000:5000 -v /home/tonydja/formation/ci-cd/Docker-API-webapp/simple_api/student_age.json:/data/student_age.json api-service
```

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/e12bb5924d71eef708ffdec59bfae04766c943ea/screenshots/docker-run.png)


<strong>- Tester le conteneur</strong>
```
curl -u toto:python -X GET http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages
```

exemple : `curl -u toto:python -X GET http://localhost:5000/pozos/api/v1.0/get_student_ages`

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/e12bb5924d71eef708ffdec59bfae04766c943ea/screenshots/curl-api.png)

<strong>- Supprimer le conteneur</strong>

Ensuite supprimer le conteneur pour pouvoir le déployer avec le frontend en utilisant le docker-compose.yml
```
docker container stop container-api-service
docker container rm container-api-service
```

## Déploiement

Pour déployer le frontend et l'API en backend nous allons exécuter le fichier docker-compose.yml

<strong>Il contient 2 services :</strong>

=> frontend<br/>
=> api

Le service frontend dépend du service api. Il ne peut pas démarrer si l'API n'est pas encore active.<br/>
Il est exposé sur le port 80.

=> depends_on:<br/>
      - api


<strong>Frontend, modification du fichier index.php :</strong>

Afin que le Frontend puisse envoyer les requêtes à l'API, il faudra spécifier l'adresse de l'API dans la variable "$url"
Nous pouvons utiliser le nom du conteneur Docker embarquant l'API, car en ayant créer notre propre réseau interne "student_network" dans le Docker-compose, celui-ci effectuera la résolution DNS par défaut.

`$url = 'http://api:5000/pozos/api/v1.0/get_student_ages`

<br/>
<strong>- Lancement de l'application</strong>
<br/><br/>

```
docker compose up
```

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/ec950f89772dab7f02253184a910a7dd3127a693/screenshots/docker-compose-up.png)


Vérifiez que l'API est bien accessible via cette URL :<br/>
http://localhost:5000/pozos/api/v1.0/get_student_ages

Pour accéder à l'application (frontend) :<br/>
http://localhost


![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/929ee6c621a724e0a673456cd359702c65d411f7/screenshots/frontend.png)



## Création d'un Registre privé

Afin d'enrichir notre application, nous allons mettre en place notre propre registre privé afin de stocker nos images en interne sans passer par une solution externe comme DockerHub.

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/a0ffd563d63d6bee7210d6bdb3118256fbcf4577/screenshots/docker-private-registry.png)


Pour cela nous ajouterons une interface graphique "UI" afin de visualiser confortablement notre bibliothèque sur un navigateur Web.

Nous utiliserons ce projet que nous allons intégrer à notre fichier docker-compose.yml <br/>
`https://hub.docker.com/r/joxit/docker-registry-ui/`


<strong>Il contient 2 services :</strong>

=> registry-server<br/>
=> registry-ui

Par défaut, ils sont exposés sur le port 5000 pour le serveur et 80 par la partie UI.
Notre application utilisant déjà ces ports sur la machine Hôte, nous allons les rediriger afin de ne pas créer de conflits.

=> registry-server => ports 9000:5000<br/>
=> registry-ui => ports 3000:80

Le service registry-ui dépend du service registry-server. l'UI ne peut pas démarrer si le serveur n'est pas encore actif.<br/>

=> depends_on:<br/>
      - registry-server


<strong>- Lancement de l'application avec le registre privé</strong>

En premier lieu nous allons supprimer l'environnement précédent et relancer le docker compose

```
docker compose down
docker compose up
```

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/8700e004ea60f6b27a8f257c0aee7e68a78a89b8/screenshots/registry.png)



<strong>- Checker les containers</strong>

Nous vérifions que nos 4 containers soient bien lancés avec un status "UP"

```
docker ps
```

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/cd517afb6c721b98327431371ea3e25380db63e2/screenshots/docker-ps.png)



<strong>- Pusher nos images Docker sur notre Registry privé</strong>

L'interface graphique de notre Registry est accessible à cette adresse :<br/>
`http://localhost:3000`


Pour Pusher une image, il faut tout d'abord Taguer l'image avec la balise de notre Registry.<br/>
Le service registry-server est exposé sur le port 9000 de notre machine Hôte. Il pointe ensuite sur le port 5000 du container Docker.<br/>

```
docker tag api-service:latest localhost:9000/api-service:latest
```

Ensuite nous utilisons la commande PUSH

```
docker push localhost:9000/api-service:latest
```


<strong>- Afficher notre Registry</strong>

=> http://localhost:3000



![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/e1af07224a33749102058d4d85ffa03d700a0e9b/screenshots/registry-ui.png)

<div align="center">

![screen](https://github.com/Tony-Dja/Docker-API-webapp/blob/e1af07224a33749102058d4d85ffa03d700a0e9b/screenshots/enjoy.jpg)

</div>

