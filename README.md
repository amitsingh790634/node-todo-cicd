================================================
File: README.md
================================================
# node-todo-cicd

Run these commands:


`sudo apt install nodejs`


`sudo apt install npm`


`npm install`

`node app.js`

or Run by docker compose

test



================================================
File: Dockerfile
================================================

FROM node:12-alpine

WORKDIR /node

COPY . .

RUN npm install
RUN npm run test

EXPOSE 8000

CMD ["node","app.js"]




================================================
File: Jenkinsfile
================================================
pipeline{
    agent { label 'dev-server' }
    
    stages{
        stage("Code Clone"){
            steps{
                echo "Code Clone Stage"
                git url: "https://github.com/LondheShubham153/node-todo-cicd.git", branch: "master"
            }
        }
        stage("Code Build & Test"){
            steps{
                echo "Code Build Stage"
                sh "docker build -t node-app ."
            }
        }
        stage("Push To DockerHub"){
            steps{
                withCredentials([usernamePassword(
                    credentialsId:"dockerHubCreds",
                    usernameVariable:"dockerHubUser", 
                    passwordVariable:"dockerHubPass")]){
                sh 'echo $dockerHubPass | docker login -u $dockerHubUser --password-stdin'
                sh "docker image tag node-app:latest ${env.dockerHubUser}/node-app:latest"
                sh "docker push ${env.dockerHubUser}/node-app:latest"
                }
            }
        }
        stage("Deploy"){
            steps{
                sh "docker compose down && docker compose up -d --build"
            }
        }
    }
}


================================================
File: app.js
================================================
const express = require('express'),
    bodyParser = require('body-parser'),
    // In order to use PUT HTTP verb to edit item
    methodOverride = require('method-override'),
    // Mitigate XSS using sanitizer
    sanitizer = require('sanitizer'),
    app = express(),
    port = 8000

app.use(bodyParser.urlencoded({
    extended: false
}));
// https: //github.com/expressjs/method-override#custom-logic
app.use(methodOverride(function (req, res) {
    if (req.body && typeof req.body === 'object' && '_method' in req.body) {
        // look in urlencoded POST bodies and delete it
        let method = req.body._method;
        delete req.body._method;
        return method
    }
}));


let todolist = [];

/* The to do list and the form are displayed */
app.get('/todo', function (req, res) {
        res.render('todo.ejs', {
            todolist,
            clickHandler: "func1();"
        });
    })

    /* Adding an item to the to do list */
    .post('/todo/add/', function (req, res) {
        // Escapes HTML special characters in attribute values as HTML entities
        let newTodo = sanitizer.escape(req.body.newtodo);
        if (req.body.newtodo != '') {
            todolist.push(newTodo);
        }
        res.redirect('/todo');
    })

    /* Deletes an item from the to do list */
    .get('/todo/delete/:id', function (req, res) {
        if (req.params.id != '') {
            todolist.splice(req.params.id, 1);
        }
        res.redirect('/todo');
    })

    // Get a single todo item and render edit page
    .get('/todo/:id', function (req, res) {
        let todoIdx = req.params.id;
        let todo = todolist[todoIdx];

        if (todo) {
            res.render('edititem.ejs', {
                todoIdx,
                todo,
                clickHandler: "func1();"
            });
        } else {
            res.redirect('/todo');
        }
    })

    // Edit item in the todo list 
    .put('/todo/edit/:id', function (req, res) {
        let todoIdx = req.params.id;
        // Escapes HTML special characters in attribute values as HTML entities
        let editTodo = sanitizer.escape(req.body.editTodo);
        if (todoIdx != '' && editTodo != '') {
            todolist[todoIdx] = editTodo;
        }
        res.redirect('/todo');
    })
    /* Redirects to the to do list if the page requested is not found */
    .use(function (req, res, next) {
        res.redirect('/todo');
    })

    .listen(port, function () {
        // Logging to console
        console.log(`Todolist running on http://0.0.0.0:${port}`)
    });
// Export app
module.exports = app;


================================================
File: docker-compose.yaml
================================================
version: '3.9'

services:
  web:
    image: trainwithshubham/node-app:latest
    ports:
      - "8000:8000"


================================================
File: package.json
================================================
{
  "name": "my-todolist",
  "version": "0.1.0",
  "dependencies": {
    "body-parser": "^1.16.0",
    "ejs": "^2.5.5",
    "express": "^4.14.0",
    "method-override": "^3.0.0",
    "sanitizer": "^0.1.3"
  },
  "scripts": {
    "start": "node app.js",
    "test": "mocha --recursive --exit",
    "sonar": "sonar-scanner"
  },
  "author": "riaan@entersekt.com",
  "description": "Basic to do list exercise",
  "devDependencies": {
    "chai": "^4.2.0",
    "mocha": "^6.2.1",
    "nyc": "^14.1.1",
    "supertest": "^4.0.2"
  }
}


================================================
File: sonar-project.properties
================================================
# required metadata
sonar.projectKey=node-todo-app
sonar.projectName=Node application
sonar.projectVersion=1.0.0
 
# optional description
sonar.projectDescription=This project demonstrates a simple node js.
 
# path to source directories
sonar.sources=./
 
# The value of the property must be the key of the language.
sonar.language=js
 
# Encoding of the source code
sonar.sourceEncoding=UTF-8


================================================
File: test.js
================================================
// Requiring module
const assert = require('assert');

// We can group similar tests inside a describe block
describe("Simple Calculations", () => {
before(() => {
	console.log( "This part executes once before all tests" );
});

after(() => {
	console.log( "This part executes once after all tests" );
});
	
// We can add nested blocks for different tests
describe( "Test1", () => {
	beforeEach(() => {
	console.log( "executes before every test" );
	});
	
	it("Is returning 5 when adding 2 + 3", () => {
	assert.equal(2 + 3, 5);
	});

	it("Is returning 6 when multiplying 2 * 3", () => {
	assert.equal(2*3, 6);
	});
});

describe("Test2", () => {
	beforeEach(() => {
	console.log( "executes before every test" );
	});
	
	it("Is returning 4 when adding 2 + 3", () => {
	assert.equal(2 + 3, 5);
	});

	it("Is returning 8 when multiplying 2 * 4", () => {
	assert.equal(2*4, 8);
	});
});
});


================================================
File: DevSecOps/README.md
================================================
# End to End DevSecOps Project for DevOps Engineers

### In this project, we will learn about  DevOps and DevSecOps tools in one project:

### Tools Covered:
-  Linux
-  Git and GitHub
-  Docker
-  Docker-compose
-  Jenkins CI/CD
-  SonarQube
-  OWASP
-  Trivy 

#

## Pre-requisites to implement this project:

-  AWS EC2 instance (Ubuntu) with instance type t2.large and root volume 29GB.

-  Jenkins installed <br>
    - Reference: <b><a href="https://www.jenkins.io/doc/book/installing/linux/#long-term-support-release"><u> Jenkins installation </a></u></b>

-  Docker and docker-compose installled
```bash
    sudo apt-get update
    sudo apt-get install docker.io -y
    sudo apt-get install docker-compose -y
```

- Trivy installed <br>
    - Reference: <b> <a href="https://github.com/DevMadhup/Trivy_Installation_and_implementation/blob/main/README.md"><u>Trivy Installation</a></u></b>

- SonarQube Server installed
```bash
    docker run -itd --name sonarqube-server -p 9000:9000 sonarqube:lts-community
```
#
## Steps for Jenkins CI/CD:

1)  Access Jenkins UI and setup Jenkins

![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/1eec417e-95ab-4497-ad31-443ecd6b999e)

#

2)  Plugins Installation:

    - Go to <b><i><u>Manage Jenkins</u></i></b>, click on <b><i><u>Plugins</u></i></b> and install all the plugins listed below, we will require for other tools integration:

        - SonarQube Scanner (Version2.16.1)
        - Sonar Quality Gates (Version1.3.1)
        - OWASP Dependency-Check (Version5.4.3)
        - Docker (Version1.5)
#

3) Go to SonarQube Server and create token

    - Click on <b><i><u> Administration </u></i></b> tab, then <b><i><u> Security </u></i></b>, then <b><i><u> Users </u></i></b> and create Token.
    -  Create a webhook to notify Jenkins that Quality gates scanning is done. (We will need this step later)

        - Go to SonarQube Server, then <b><i><u> Administration </u></i></b>, then <b><i><u> Configuration </u></i></b> and click on <b><i><u> Webhook </u></i></b>, add webhook in below <b>Format</b>:
        > http://<jenkins_url>:8080/sonarqube-webhook/
        
        Example: 
        
        ```bash
            http://34.207.58.19:8080/sonarqube-webhook/
        ```

        ![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/b9ef2301-b8ff-46f4-a457-6345d5e2dab6)


        ![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/08a33164-f6a6-4c5d-8a34-7091cf8a5745)

#

4) Go to Jenkins UI <b><i><u> Manage Jenkins </u></i></b>, then <b><i><u> Credentials </u></i></b> and add SonarQube Credentials.

![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/f6db72ec-7d8c-4f4c-ae7a-55d99dd20ce9)

#

5) Now, It's time to integrate SonarQube Server with Jenkins, go to <b><i><u> Manage Jenkins </u></i></b>, then <b><i><u> System </u></i></b> and look for <b><i><u> SonarQube Servers </u></i></b> and add SonarQube.

![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/54849cb2-fe56-4acd-972d-3057a0eb3deb)

#

6) Go to <b><i><u> Manage Jenkins </u></i></b>, then <b><i><u> tools </u></i></b>, look for <b><i><u> SonarQube Scanner installations </u></i></b> and add SonarQube Scanner.

> Note: Add name as ```Sonar```

![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/1fe926f6-a844-42d4-bce4-62193dde6640)

7) Integrate OWASP with Jenkins, go to <b><i><u> Manage Jenkins </u></i></b>, then <b><i> tools </i></b>, look for <b><i><u>Dependency-Check installations</u></i></b> and add Dependency-Check.

> Note: Add name as ```dc```

![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/14516995-0c96-4110-bb96-97a37a9fe57d)

#

8) For trivy, we have already installed it, in pre-reuisites.

![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/0fcd1620-bd64-4286-bc13-f6652d4527c6)

#

9) Now, It's time to create a CI/CD pipeline in Jenkins:
    -  Click on <b><i><u>New Item</u></i></b> and give it a name and select <b><i><u>Pipeline</u></i></b>.
    -  Select <b><i><u>GitHub Project</u></i></b> and paste your GitHub repository link.
    -  Scroll down and in <b><i><u>Pipeline</u></i></b> section select <b><i><u>Pipeline script from SCM</u></i></b>, because our Jenkinsfile is present on GitHub.

    ![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/39af1b22-28aa-4e36-b98c-0e7f120b5fbf)

    ![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/b7153556-f847-40ee-9a98-ff3609930abd)
#

10) At last run the pipeline and after sometime your code is deployed using DevSecOps.

![image](https://github.com/DevMadhup/node-todo-cicd/assets/121779953/f566d980-82ee-4ad6-9ff3-cb970885560e)
#

<b>Congratulations!!! You have done it</b>

- If you find any issue during the execution of this project, let me know on LinkedIn

- Happy Learning :)
#

## These are our community Links
  <a href="https://discord.com/channels/824622549182185493/824622550327623692">
    <img width="30px" src="https://www.vectorlogo.zone/logos/discordapp/discordapp-tile.svg" />
  </a>&ensp;
    <a href="https://t.me/trainwithshubham">
    <img width="30px" src="https://www.vectorlogo.zone/logos/telegram/telegram-icon.svg" />
  </a> 
  </a>&ensp;

  <a href="https://www.linkedin.com/in/shubhamlondhe1996/">
    <img width="30px" src="https://www.vectorlogo.zone/logos/linkedin/linkedin-icon.svg" />
  </a>&ensp;

 <a href="https://www.youtube.com/@TrainWithShubham">
  <img width="30px" src="https://i.pinimg.com/originals/46/02/cb/4602cbc18967da9c1eba7452905cd99b.png" />
  </a>&ensp;

  <a href="https://chat.whatsapp.com/FvRlAAZVxUhCUSZ0Y1s7KY">
  <img width="30px" src="https://www.vectorlogo.zone/logos/whatsapp/whatsapp-icon.svg" />
</a>&ensp;



================================================
File: DevSecOps/Jenkinsfile
================================================

pipeline {
    
    agent any
    environment{
        SONAR_HOME = tool "Sonar"
    }
    stages {
        
        stage("Code"){
            steps{
                git url: "https://github.com/LondheShubham153/node-todo-cicd.git" , branch: "master"
                echo "Code Cloned Successfully"
            }
        }
        stage("SonarQube Analysis"){
            steps{
               withSonarQubeEnv("Sonar"){
                   sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=nodetodo -Dsonar.projectKey=nodetodo -X"
               }
            }
        }
        stage("SonarQube Quality Gates"){
            steps{
               timeout(time: 1, unit: "MINUTES"){
                   waitForQualityGate abortPipeline: false
               }
            }
        }
        stage("OWASP"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Build & Test"){
            steps{
                sh 'docker build -t node-app-batch-6:latest .'
                echo "Code Built Successfully"
            }
        }
        stage("Trivy"){
            steps{
                sh "trivy image node-app-batch-6"
            }
        }
        stage("Push to Private Docker Hub Repo"){
            steps{
                withCredentials([usernamePassword(credentialsId:"DockerHubCreds",passwordVariable:"dockerPass",usernameVariable:"dockerUser")]){
                sh "docker login -u ${env.dockerUser} -p ${env.dockerPass}"
                sh "docker tag node-app-batch-6:latest ${env.dockerUser}/node-app-batch-6:latest"
                sh "docker push ${env.dockerUser}/node-app-batch-6:latest"
                }
                
            }
        }
        stage("Deploy"){
            steps{
                sh "docker-compose down && docker-compose up -d"
                echo "App Deployed Successfully"
            }
        }
    }
}


================================================
File: k8s/deployment.yml
================================================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app-deployment
  namespace: node-app
  labels:
    app: node-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      name: node-pod
      namespace: node-app
      labels:
        app: node-app
    spec:
      containers:
        - name: node-container
          image: trainwithshubham/node-app-batch-6
          ports:
            - containerPort: 8000
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"


================================================
File: k8s/pod.yml
================================================
apiVersion: v1
kind: Pod
metadata:
  name: node-pod
  namespace: node-app

spec:
  containers:
    - name: node-container
      image: trainwithshubham/node-app-batch-6
      ports:
        - containerPort: 8000


================================================
File: k8s/replica-sets.yml
================================================
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: node-app-replica-set
  namespace: node-app
  labels:
    app: guestbook
    tier: node-label
spec:
  # modify replicas according to your case
  replicas: 6
  selector:
    matchLabels:
      tier: node-label
  template:
    metadata:
      namespace: node-app
      labels:
        tier: node-label
    spec:
      containers:
      - name: node-container-rep
        image: trainwithshubham/node-app-batch-6


================================================
File: k8s/service.yml
================================================
apiVersion: v1
kind: Service
metadata:
  name: node-app-service
  namespace: node-app
spec:
  type: NodePort
  selector:
    app: node-app
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30003


================================================
File: kustomize/README.md
================================================
# Google Kubernetes Engine (GKE) Autopilot and Kustomize

Welcome to the NodeJS DevOps Project GitHub repository! This repository contains instructions and resources to help you get started with Google Kubernetes Engine (GKE) Autopilot and Google Cloud Shell. 

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
  - [Install HELM](#install-helm)
  - [Install nginx-controller](#install-nginx-controller)
- [Kubernetes Autopilot](#kubernetes-autopilot)
  - [Create a Namespace](#create-a-namespace)
  - [Deploy an Application](#deploy-an-application)
- [Google Cloud Shell](#google-cloud-shell)
- [Useful Commands](#useful-commands)

## Introduction
This repository provides a step-by-step guide to setting up a Kubernetes environment with GKE Autopilot and using Google Cloud Shell for management and deployment tasks.

## Prerequisites
Before you begin, make sure you have the following prerequisites:
- A Google Cloud Platform (GCP) account with billing enabled.
- Google Cloud SDK (gcloud) installed on your local machine.
- Access to a GKE cluster with Autopilot enabled.


# Google Cloud Shell
Google Cloud Shell provides a browser-based shell environment that you can use for managing your GCP resources and interacting with Kubernetes clusters.

To access Google Cloud Shell, follow these steps:

Log in to your GCP Console.
Click on the "Activate Cloud Shell" button in the upper-right corner.

### Install HELM
To install HELM, use the following commands:

## Add & Update the HELM repository
```bash

helm repo add helm https://helm.sh/helm

helm repo update
```
## Install nginx-controller
To install the nginx-controller, use the following commands:

## Add & Update the nginx-controller repository
```bash
helm repo add nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx nginx/ingress-nginx 
helm search repo nginx
```
## List all ValidatingWebhookConfigurations in all namespaces
```bash
kubectl get ValidatingWebhookConfiguration -A
```
## Delete a ValidatingWebhookConfiguration (e.g., nginx-ingress-nginx-admission)
```bash
kubectl delete -A ValidatingWebhookConfiguration nginx-ingress-nginx-admission
```
# On Shell
## Create a Namespace
You can create a Kubernetes namespace using the following command:

``` bash
kubectl create namespace <namespace-name>
```
# Deploy an Application using Kustomize

To deploy an application, apply the YAML configuration file to your namespace:

```bash

kubectl apply -k <path-to-overlays-environment> 
```

# Useful Commands
Here are some useful commands to help you manage your Kubernetes environment:

``` bash=
# List all pods in the current namespace
kubectl get pods -n <>namespace>

# Create a new Kubernetes namespace (e.g., "dev" or "sit")
kubectl create ns <namespace-name>

# Apply a YAML configuration file to create resources
kubectl apply -f <path-to-yaml-file>
```

If you have any questions or encounter issues, please don't hesitate to open an issue or reach out to the community for assistance.

Happy learning! 🚀


================================================
File: kustomize/base/app-1/app-1.yml
================================================
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app-1
  name: app-1-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-1
  template:
    metadata:
      labels:
        app: app-1
    spec:
      containers:
      - name: app-1-containter
        image: trainwithshubham/node-app-test-new:latest
        imagePullPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: app-1
  name: app-1-service
  namespace: default
spec:
  type: NodePort
  ports:
  - name: webport
    port: 8000
    targetPort: 8000
  selector:
    app: app-1   


================================================
File: kustomize/base/app-1/kustomization.yml
================================================
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
metadata:
  name: app-1-mapping

resources:
  - app-1.yml


================================================
File: kustomize/base/ingress/ingress.yml
================================================
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: default
spec:
  ingressClassName: "nginx"
  rules:
  - host: invalid.demo.trainwithshubham.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service: 
            name: app-1-service
            port: 
                number: 8000     


================================================
File: kustomize/base/ingress/kustomization.yml
================================================
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
metadata:
  name: ingress-mapping

resources:
  - ingress.yml


================================================
File: kustomize/overlays/dev/dev-ingress-patch.json
================================================
[
    {
        "op": "replace",
        "path": "/spec/rules/0/host",
        "value": "dev.demo.trainwithshubham.com"
    }
]


================================================
File: kustomize/overlays/dev/kustomization.yml
================================================
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
metadata:
  name: dev-mapping

namespace: dev
namePrefix: dev-
replicas:
- name: app-1-deployment
  count: 1

resources:
  - ../../base/app-1/
  - ../../base/ingress/

patches:
- target:
    kind: Ingress
    name: app-ingress
  path: dev-ingress-patch.json


================================================
File: kustomize/overlays/prd/kustomization.yml
================================================
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
metadata:
  name: prd-mapping

namespace: prd
namePrefix: prd-

replicas:
- name: app-1-deployment
  count: 2

resources:
  - ../../base/app-1/
  - ../../base/ingress/


patches:
- target:
    kind: Ingress
    name: app-ingress
  path: prd-ingress-patch.json


================================================
File: kustomize/overlays/prd/prd-ingress-patch.json
================================================
[
    {
        "op": "replace",
        "path": "/spec/rules/0/host",
        "value": "prd.demo.trainwithshubham.com"
    }
]


================================================
File: terraform/main.tf
================================================
resource "docker_image" "todo_image" {
	name = "trainwithshubham/todo-app-node:latest"
	keep_locally = false
}

resource "docker_container" "todo_container" {
	image = docker_image.todo_image.name
	name = "todoapp-container"
	ports {
		internal = 8000
		external = 8000
	}

	depends_on = [
		docker_image.todo_image
	]
	
}


================================================
File: terraform/terraform.tf
================================================
terraform {
	required_providers {
	docker = {
	source  = "kreuzwerker/docker"
	version = "3.0.2"
}
}
}


================================================
File: views/edititem.ejs
================================================
<!DOCTYPE html>

<html>

    <head>
        <title>Edit <%- todo %></title>
        <style>
            a {
                text-decoration: none;
                color: black;
            }

        </style>
    </head>

    <body>
        <h1>Edit <%- todo %>:</h1>
        <form action="/todo/edit/<%= todoIdx %>" method="post">
            <p>
                <label for="editTodo">Change to:</label>
                <input type="hidden" name="_method" value="PUT">
                <input type="text" name="editTodo" id="editTodo" autofocus placeholder="<%- todo %>" />
                <input type="submit" value="Save" />
            </p>
        </form>
    </body>

</html>


================================================
File: views/todo.ejs
================================================
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Todo List APP test</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                background-color: #f4f4f4;
                color: #333;
                margin: 0;
                padding: 0;
            }

            h1 {
                background-color: #4CAF50;
                color: white;
                margin: 0;
                padding: 20px;
                text-align: center;
            }

            ul {
                list-style: none;
                padding: 0;
            }

            ul li {
                background: #fff;
                border: 1px solid #ddd;
                margin-bottom: 10px;
                padding: 10px;
                position: relative;
            }

            ul li a {
                color: #333;
                text-decoration: none;
                font-weight: bold;
            }

            ul li a:hover {
                color: #4CAF50;
            }

            .delete-btn {
                color: red;
                position: absolute;
                right: 10px;
                top: 10px;
            }

            .edit-btn {
                color: #FFC107;
                position: absolute;
                right: 40px;
                top: 10px;
            }

            form {
                background: #fff;
                padding: 20px;
                margin-top: 20px;
            }

            label {
                display: block;
                margin-bottom: 10px;
            }

            input[type="text"] {
                width: 100%;
                padding: 10px;
                margin-bottom: 10px;
                border: 1px solid #ddd;
            }

            input[type="submit"] {
                background-color: #4CAF50;
                color: white;
                border: none;
                padding: 10px 20px;
                cursor: pointer;
            }

            input[type="submit"]:hover {
                background-color: #45a049;
            }
        </style>
    </head>

    <body>
        <h1>Hello Junoon Batch 8 (Jenkins), Write your plan on Learning Jenkins</h1>
        <ul>
            <% todolist.forEach(function(todo, index) { %>
            <li>
                <a href="/todo/delete/<%= index %>" class="delete-btn">✘</a>
                <a href="/todo/<%= index %>" class="edit-btn">✎</a>
                <%- todo %>
            </li>
            <% }); %>
        </ul>

        <form action="/todo/add/" method="post">
            <p>
                <label for="newtodo">What should I do?</label>
                <input type="text" name="newtodo" id="newtodo" autofocus />
                <input type="submit" value="Add" />
            </p>
        </form>
    </body>
</html>


