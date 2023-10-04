
<div align="center">

![Docker Build](https://img.shields.io/badge/docker-build-green)       ![Docker Build](https://img.shields.io/badge/dockercompose-up-green)            ![example workflow](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/actions/workflows/github-ci.yml/badge.svg)         

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


# Deploy containers

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
Then enter the password and validate.

```
docker exec -it jenkins-server /bin/bash
```

inside the container :

```
cat /var/jenkins_home/secrets/initialAdminPassword 
```

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/944cb13af42eac81d10ddbc4ad488fad5fb15c6f/screenshots/initialpassword.png)


It' OK, so now create a user and confirm the IP address with the open port.

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/9fa25e562d7ff12208cf3d6b45e0926afb985eb8/screenshots/first-user.png)

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/9fa25e562d7ff12208cf3d6b45e0926afb985eb8/screenshots/confirm-ip.png)



# Create Jenkins project

We are going to create a Jenkins pipeline to Build a simple HTML5 website.
You can use my repository or take yours.

```
https://github.com/Tony-Dja/Docker-contenerized-website
```


Enter a name for your project

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/418a4415968e94efc1682b142679780c0bd98689/screenshots/project.png)


Click on checkbox "Build with parameters", and add 2 variables with string type :

      - IMAGE_NAME => name of Docker image you are going to build
      - IMAGE_TAG => Tag of the image


![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/418a4415968e94efc1682b142679780c0bd98689/screenshots/string1.png)

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/418a4415968e94efc1682b142679780c0bd98689/screenshots/string2.png)


From the Build Steps section, select "execute shell script"

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/418a4415968e94efc1682b142679780c0bd98689/screenshots/shell.png)


And last, add script to clone the repo, and Build image.
Then validate project

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/418a4415968e94efc1682b142679780c0bd98689/screenshots/script.png)




# Run pipeline

To run pipeline, click on "Build with parameters" and "Build"

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/418a4415968e94efc1682b142679780c0bd98689/screenshots/string2.png)


The job is running, you can watch job in progress 

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/418a4415968e94efc1682b142679780c0bd98689/screenshots/string2.png)


The job is finished

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/418a4415968e94efc1682b142679780c0bd98689/screenshots/string2.png)




# Blue Ocean plugin

In the Dockerfile, we install Blue Ocean plugin. It allows to visualize a graphic interface for our pipeline.<br/>
you can find the link in Jenkins navigation menu 

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/418a4415968e94efc1682b142679780c0bd98689/screenshots/string2.png)

![screen](https://github.com/Tony-Dja/Jenkins_CI-CD_pipeline/blob/418a4415968e94efc1682b142679780c0bd98689/screenshots/string2.png)



