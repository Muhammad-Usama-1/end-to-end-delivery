## Installing SonarQube Locally

For running Sonarqube we requred JAVA and database server (postgress and db name sonarqube)

Install sonarqube and extract the folder then find conf and wrapper file file insde the director

In wrapper give the path of your java
and now open sonar.properties file to configure dtabase username and password correctly in order to connect to your local postgress

find the script inside bin and run

```bash
./sonar.sh
```

## Installing Sonarqube fron one line

you must have docker and docker-compose install and also we need to make some change in
/etc/sysctl.conf

add these two line at the bottom of the file /etc/sysctl.conf
vm.max_map_count=262144
fs.file-max=65536

### What is Docker-compose

It is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. Since Docker Compose lets you configure related containers in a single YAML file, you get the same Infrastructure-as-Code abilities as Kubernetes. But they come in a simpler system that’s more suited to smaller applications that don’t need Kubernetes’ resiliency and scaling.

The purpose of docker-compose is to function as docker cli but to issue multiple commands much more quickly. To make use of docker-compose, you need to encode the commands you were running before into a docker-compose.yml file

Run docker-compose up and Compose starts and runs your entire app.
if you dont have docker install first install docker and docker-compose from these commands

```bash
sudo apt-get update
sudo apt install docker.io -y
sudo chmod 666 /var/run/docker.sock
# Installing Docker compose
sudo apt install docker-compose -y
```

```yaml
version: "3"
services:
  sonarqube:
    image: sonarqube:lts-community
    container_name: sonarqube
    restart: unless-stopped
    environment:
      - SONARQUBE_JDBC_USERNAME=sonar
      - SONARQUBE_JDBC_PASSWORD=password123
      - SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonarqube
    ports:
      - "9000:9000"
      - "9092:9092"
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_bundled-plugins:/opt/sonarqube/lib/bundled-plugins

  db:
    image: postgres:12
    container_name: db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=password123
      - POSTGRES_DB=sonarqube
    volumes:
      - sonarqube_db:/var/lib/postgresql10
      - postgresql_data:/var/lib/postgresql10/data

volumes:
  postgresql_data:
  sonarqube_bundled-plugins:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_db:
  sonarqube_extensions:
```

```bash
docker-compose up -d && docker-compose logs -f
```

Now SonarQube is up and running now we will setup jenkins in the same way (docker-compose)

create a directory jenkins or any name inside create a file docker-compose.yaml

```bash
 docker network create -d bridge devnetwork
```

```yaml
version: "3.5"
services:
  jenkins:
    container_name: jenkins-docker
    image: jenkins/jenkins:lts
    ports:
      - 8080:8080
    networks:
      - devnetwork
networks:
  devnetwork:
    name: devnetwork
```

```bash
#for check config
docker-compose config
```

check password in var/lib/jenkins/admintrativepassword
and install suggested plugin
after setting up user install two addition plugin nodejs and sonarqube scanner for scanning node js application

```jenkins
pipeline {
    agent any
    tools  {
        // This nodejs name should be same as you configure in jenkins
        nodejs "17.3.1"
    }

    stages {
        stage('Hello') {
            steps {
                sh "npm version"
            }
        }
        stage('CheckoutCode'){
            steps {
             git  url: 'https://github.com/Muhammad-Usama-1/nodejs-app-mss'
            }

        }
        stage("Building the Applcation") {
            steps {
                sh "npm install"
            }
        }

         stage('Executating the Sonarqube report') {
            steps {
                sh "npm run sonar"
            }
        }
    }
}
```
