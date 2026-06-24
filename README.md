# Jenkins
## We can install jenkins from official documentation of Jenkins as it is based on Java so first we need to install Java. [Jenkins Installation](https://www.jenkins.io/doc/book/installing/linux/) Generally Jenkins run on Port 8080 so make sure to add port in Inbound Securities.
<img width="806" height="399" alt="Screenshot 2026-06-23 210432" src="https://github.com/user-attachments/assets/cd3bc147-30bd-44c1-8182-79941c630dd0" />

<img width="857" height="342" alt="Screenshot 2026-06-23 210653" src="https://github.com/user-attachments/assets/68c972f2-9ce5-4b7f-bba6-7515b879ea01" />

## There are various types in which we can create job in Jenkins such as: Freestyle Project(where we can do anything like code clone, docker run, we have to give commands one after other for this), Pipeline(in this we have to define Declarative Pipelines that we discuss later), Multi-configuration Project(in this if we want to write in multiple environment like dev,staging, production), Folder(is basically for grouping things we can write anything inside that folder), MultiBranch Pipeline(in git we have multiple branch such as master,dev,production so we can perform for them) and in Organization Folder we can keep different jobs in that folder.

## Freestyle Project
<img width="778" height="434" alt="Screenshot 2026-06-23 214813" src="https://github.com/user-attachments/assets/5412609d-9547-4828-8431-2dc1fe5df4b8" />

<img width="662" height="445" alt="Screenshot 2026-06-23 211447" src="https://github.com/user-attachments/assets/a435c0ed-260f-4d3d-80b5-3a36f055a74b" />


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

<img width="959" height="263" alt="Screenshot 2026-06-23 214839" src="https://github.com/user-attachments/assets/c6ad5002-7373-4f33-a073-db7117472957" />
![Uploading Screenshot 2026-06-23 214813.png…]()


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

<img width="575" height="380" alt="Screenshot 2026-06-23 215623" src="https://github.com/user-attachments/assets/1dbd1c04-713d-4b23-bef9-1a02a8563d6a" />


# Shared library:
## It is groovy code that we can reuse in our pipeline
## create vars folder in your repo and inside that keep all library code
```groovy
basic file(clone.groovy)

def call(String url,String branch){
    echo "Cloning the code"
    git url:"${url}",branch:"${branch}"
}
``` 
## after creating all this files, do this in jenkins so that it will know where your shared libraries are there
## Manage Jenkins -> System -> Global Trusted Pipeline Libraries -> Add(and add proper name of shared library and from where we are getting this like github )

## after this make sure to add this at the top of pipeline
```groovy
@Library("<here library name comes that we gave while adding pipeline libraries>") _

like this 
@Library("Shared") _

and in stage you can use like this

stage("Code"){
    steps{
        script{
            clone("url to be clones","main")
        }
    }
}
```

## instead of changing jenkins script everytime in jenkins ui we can create one file "JenkinsFile" and we can put code there and in Jenkins ui we can select Pipeline Script from SCM and can provide the github link where this file is present

# Role Management in Jenkins:
## create user in Jenkins and install plugin Role-based Authorization strategy so that we can manage roles
## Manage Jenkins -> Security -> Authorization ->(select Role Based Strategy) and Save and then inside Manage Jenkins -> we can see a option (Manage and Assign Roles) so we can add role and can manage what all access we want and then assign that role to user
