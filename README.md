# Jenkins
## We can install jenkins from official documentation of Jenkins as it is based on Java so first we need to install Java. [Jenkins Installation](https://www.jenkins.io/doc/book/installing/linux/) Generally Jenkins run on Port 8080 so make sure to add port in Inbound Securities.


## There are various types in which we can create job in Jenkins such as: Freestyle Project(where we can do anything like code clone, docker run, we have to give commands one after other for this), Pipeline(in this we have to define Declarative Pipelines that we discuss later), Multi-configuration Project(in this if we want to write in multiple environment like dev,staging, production), Folder(is basically for grouping things we can write anything inside that folder), MultiBranch Pipeline(in git we have multiple branch such as master,dev,production so we can perform for them) and in Organization Folder we can keep different jobs in that folder.

# Declarative Pipeline:
```groovy
pipeline {
    agent any  (or we can give like that agent {label "name"})

    stages{
        stage('Hello'){
            steps{
                echo 'Hello World'
            }
        }

        ....inside this we can create multiple stages like code clone, build, test, deploy
    }
}
```

## It is not preferred to run jobs on same on which jenkins is running so we create another agent and run jobs on that agent

# Setting Up Agent:
## Inside Manage Jenkins ---> Set up Agent(so we create another server for agent and it is must that java should be installed on Agent Server)
## Jenkins will connect to its agent through SSH(using public and private key pair)
## Generate key pair in jenkins master(command:ssh-keygen), now the private key shoulb be added in Jenkins ui where we are setting up the agent and public key should be added in ~/..ssh/authorized_keys in agent server(in ui of Jenkins while setting up agent use Launch Method as "Launch Agent via SSH")
## after setting up agent we can use in our pipeline like this agent {label "label name that we provided while creating agent"}

## while creating pipeline there is one option in Build Triggers where we can enable Github Hook Trigger so that when something changes on Github then this pipeline will automatically trigger.

## For better pipeline view while building, we can install plugin from Manage Jenkins -> Plugins -> Available Plugin and then install Pipeline: Stage view

# Pipeline for cloning,building,pushing to dockerhub,deploying
```groovy
pipeline {
    agent {label "worker"}
    
    stages{
        stage("Clone Code"){
            steps {
                echo "Cloning the code"
                git url:"https://github.com/angadnagar/django-notes-app.git", branch: "main"
            }
        }
        stage("Build"){
            steps {
                echo "Building the image"
                sh "docker build -t my-note-app ."
            }
        }
        stage("Push to Docker Hub"){
            steps {
                echo "Pushing the image to docker hub"
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker tag my-note-app ${env.dockerHubUser}/my-note-app:latest"
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker push ${env.dockerHubUser}/my-note-app:latest"
                }
            }
        }
        stage("Deploy"){
            steps {
                echo "Deploying the container"
                sh "docker-compose down && docker-compose up -d"
                
            }
        }
    }
}
```

# Credentials Bindings:
## Manage Jenkins -> Credentials(Under Security Section) -> global credentials -> Add Credentials

## Now after creation of this we can use it in our pipeline like this
```groovy
stage("Push to Docker Hub"){
            steps {
                echo "Pushing the image to docker hub"
                withCredentials([usernamePassword(credentialsId:"dockerHub", credentialspasswordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker tag my-note-app ${env.dockerHubUser}/my-note-app:latest"
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker push ${env.dockerHubUser}/my-note-app:latest"
                }
            }
        }
```

# adding Github Webhook:
## go to settings of your repo(not github settings) and then from side menu click on Webhooks -> Add Webhook

