# Jenkins in docker for build docker containers in docker 

## Badges

[![](https://images.microbadger.com/badges/version/rmuhamedgaliev/jenkins.svg)](https://microbadger.com/images/rmuhamedgaliev/jenkins "Get your own version badge on microbadger.com")
[![](https://images.microbadger.com/badges/version/rmuhamedgaliev/jenkins:lts.svg)](https://microbadger.com/images/rmuhamedgaliev/jenkins:lts "Get your own version badge on microbadger.com")

![Jenkins](https://piotrminkowski.files.wordpress.com/2017/03/jenkins-docker-muscles.jpg)

## About

This image provide jenkins in docker for build docker containers in docker agents. As example project structure for build project in build containers. [Build Container pattern](https://jenkins.io/doc/book/pipeline/docker/)

```bash
.
├── Dockerfile.build
├── Jenkinsfile
```

__Dockerfile.build__ content:

```docker
FROM node:11

RUN apt update \
    && apt install -y \
        bash unzip wget python python-pip software-properties-common
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367 && \
    echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main" > /etc/apt/sources.list.d/ansible.list && \
    apt-get update && \
    apt-get install --no-install-recommends -y \
      ansible

RUN pip install boto boto3 botocore

RUN npm install yarn -g

```

__Jenkinsfile__ content:

```groovy
pipeline {

    options {
        buildDiscarder (
            logRotator (
                artifactDaysToKeepStr: '',
                artifactNumToKeepStr: '',
                daysToKeepStr: '',
                numToKeepStr: '1'
            )
        )
        disableConcurrentBuilds()
    }

    agent any

    stages {
        stage('Install dependencies') {
            agent {
                dockerfile {
                    filename 'Dockerfile.build'
                    args '-v ${PWD}:/usr/src/app -w /usr/src/app'
                    label 'build-image'
                    reuseNode true
                }
            }
            steps {
                sh 'yarn'
                sh 'cd src && yarn'
            }
        }


        stage('Run tests') {
            agent {
                dockerfile {
                    filename 'Dockerfile.build'
                    args '-v ${PWD}:/usr/src/app -w /usr/src/app'
                    label 'build-image'
                    reuseNode true
                }
            }
            steps {
                sh 'yarn test'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
```

This example provide base docker container with `node.js` and ansible for building app. This approach allow avoid prepare global environment for build app.

## Usage

For usage just create `docker-compose.yml` file and use:

```yml
version: "3"

services:
  jenkins:
    container_name: jenkins-dev
    image: rmuhamedgaliev/jenkins:lts
    restart: always
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock

volumes:
  jenkins_home:
```

and just up it `docker-compose up`.