# jenkins-notes
Notes for Jenkins a CI/ CD Tool for DevOps

# What is CI/ CD?
- One of the Approach of SDLC.

## CI (Continuous Integration)
- That means instead of deploying your code at the end of day, week or anytime you want you will deploy it the moment you commit and push it in your remote repository.
- CI includes the build, test, and merge process of your project.

## CD (Continuous Delivery)
- That means making your source code ready to be released in production/ live environment. Also having a specific version of your project ready to be delivered in your client or to your co-workers or deployed in different environments(To be discussed later).

## Why do need CI/ CD?
First we identify the manual process of how our source code will reach upto deployment.

### Steps of Build and Deployment Process
1. Commit and push your latest code in remote repository.
2. Run unit test before building your project.
3. Build your project to create war or jar file.
4. Create an docker image and push it in docker registry (Docker hub).
5. Deploy your app.

As you can see theres nothing wrong in this stepd but imagine doing it every day, every week, or anytime you want its a lot of work and labor needed.

### Challenges in Manual Process.
1. Repeated Work.
2. Takes a lot of time and effort.
3. Deploying code in multiple environment is just nightmare.
4. Error prone.

Thats why we need CI/ CD to resolved all of this increasing our productivity and less time and effort for this repeated work.

# Different Realtime Enviroments
- Development Environment
- Quality Assurance Environment
- User Assurance Test Environment
- Production Environment

# Different Type of Teams in Realtime Project
- *Development Team*: Responsible for writing project source code.
- *Quality Assurance Team*: Responsible for testing the development team delivered project and also verify and validate all the system requirements.
- *Operation Team*: Responsible for Deployment of project.
###### Development + Operation Team => DevOps

# What is Jenkins
- Free and open-source software
- Written in Java Languange
- Used for automation pf Build and Deployment process.
- Using Jenkins we can implement CI/ CD

## What is pipeline
- It is the series of steps to automate the build and deployment process of your project.
###### Its like your house water pipeline how will the water flow through the pipeline to reach the water destination.

## Jenkins Project Setup
### Source Code Management
    - Where your source code coming from (Usually Github is used in this)

### Build Triggers
     - When you want to execute and automate the build process.
     - POLL SCMM is used to continously pull and automate build process with specified cron job

### Build Environment
     - Setup the environment for build process.
 
 ### Build Steps
     - How you want to do the build?
	
 ### Post Build Actions
      - What you want to do after build process is finished either create an docker image and push it in docker registry

## Two Types of Jenkins Scripting Scripting
### Scripting Pipeline: Starts with *node* and using Groovy Languange
### Declarative Pipeline: Starts with *pipeline*

## What is jenkinsfile
```Jenkinsfile
pipeline {
  	agent any

	tools { // Add tools here
	}

	environment { // Add environment variables here to be access in this Jenkinsfile
	}

	stages { // Series of Stage to be executed to Automate the build and deployment process of your project
		stage("<stage_name>") {
			steps { // This is where your command will go
			}
		}
	}

	post {
		always {} // Execute any command no matter what happen
		success {} // Execute any command only in build success 
		failure {} // Execute any command only in build failure
	}
}
```

# Basic Jenkins Set up when you are using Java + Maven + Docker
## Installation (Only do these steps once)
1. [Install Jenkins Server](https://github.com/ashokitschool/DevOps-Documents/blob/main/01-Jenkins-Server-Setup.md)
2. [Add Maven Settings in Jenkins](https://github.com/ashokitschool/DevOps-Documents/blob/main/04-Jenkins-Docker-Project.md#step-2--configure-maven-as-global-tool-in-jenkins)
3. [Install Docker and Add User Priviledges](https://github.com/ashokitschool/DevOps-Documents/blob/main/04-Jenkins-Docker-Project.md#step-3--setup-docker-in-jenkins)

## Jenkins Job Environment Setup (Only do these steps once)
4. Create DockerHub Credential(Access Token) for jenkins. The idea is for jenkins to have permission to access your dockerhub account
   - Goto Your DockerHub Account > Click your Profile > My Account > Security > Click New Access Token > Provide the required fields > Click Generate > Click copy and close.
5. Add the DockerHub Credentials(Access Token) in Jenkins Server
   - Goto Your Jenkins Webb App > Manage Jenkins > Under Security Tab (Click Credentials) > Under Stores scoped to Jenkins (Click System) > Click Global credentials (unrestricted) > Click Add Credential > Add Username(Your DockerHub Username), Add Password (Your DockerHub Credential Access Token), Add ID (You will use this ID to access your credentials in Jenkins pipeline code) > Click Create
  
## Jenkins Job Setup 
1. Create a Jenkins Pipeline
2. New Item
3. Provide Job Name and Click Pipeline
4. Add GitHub project link
5 Under Build Trigger > Check POLL SCM (Everytime you push in specified repo this job will be triggereed automatically) and supply the cron expression.
6. Add Pipeline Script (I will use my own api the security-question-api as example).
```Jenkinsfile
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.6' // Maven 3.9.6 is the name of maven I previously provide in setting up maven in jenkins 
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-access-token-for-jenkins') // docker-hub-access-token-for-jenkins is the ID of my previously created credential
    }
    
    stages {
        stage("Clone Security Question API from Github") {
            steps {
                echo "Cloning Security Question API from Github. Please Wait..."
                git branch: 'main', 
                    url: 'https://github.com/Elleined/security-question-api'
                echo "Cloning Security Question API from Github. Success!"
            }
        }
        
        stage("Build project using maven") {
            steps {
                echo "Cleaning and Generating jar file using maven. Please Wait..."
                sh 'mvn clean install'
                echo "Cleaning and Generating jar file using maven. Success!"
            }
        }
        
        stage("Create Docker Image") {
            steps {
                echo "Creating docker image. Please Wait..."
                sh 'docker build -t sqa:latest .'
                echo "Creating docker image. Success!"
            }
        }
        
        stage("Log in to DockerHub") {
            steps {
                echo "Logging in to DockerHub. Please Wait..."
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                echo "Logging in to DockerHub. Success!"
            }
        }
        
        stage("Push docker image to DockerHub") {
            steps {
                echo "Pushing docker image to DockerHub. Please Wait..."
                sh 'docker tag sqa:latest elleined/sqa:latest'
                sh 'docker push elleined/sqa:latest'
                echo "Pushing docker image to DockerHub. Success!"
            }
        }
        
    } // End of Stages
    
    post {
        always {
            sh 'docker logout'
        }
    }
    
} // End of Pipeline
```

# Backup and Restore
## Installation
1. Go to Manage Jenkins > Click Plugins > Available Plugins
3. Search ThinBackup and install
4. Restart the jenkins server by http://<jenkins_server_ip>:<jenkins_server_port>/restart
5. Go to Manage Jenkins > Under Tools and Action > Click ThinBackup
6. Click Settings
7. Fill out the required configuration fields to backup and restore jenkins server (!!!Note backup directory must be */var/lib/jenkins/backup* to avoid headache)

## Backup
- Usually backups runs automatically depends on cron expression you set in thin backup settings. Otherwise to run backup manually heres the steps.
1. Go to Manage Jenkins > Under Tools and Action > Click ThinBackup
2. Click Backup Now and that's it

## Restore
1. Go to Manage Jenkins > Under Tools and Action > Click ThinBackup
2. Click Restore and Select the backup you want to restore.
3. Lastly restart the jenkins server to reflect the restored backup
4. Restart the jenkins server by http://<jenkins_server_ip>:<jenkins_server_port>/restart

# Most common pipeline
1. Run test cases
2. Build the project
3. Build a image // Dockerfile
4. Login in docker
5. Push the image
6. SSH in server (down and pull and run docker compose)
7. Run the image
8. Clean up

# Jenkins Credentials

## Stores (Who)
- Who can use it?
- Gives structure to your creds

1. System
	- Jenkins itself
	- Agents
2. Global
	- Usable by all jobs

3. Folder

## Domains (Where)
- Where it can be used?
- Gives safety to your creds


# For more Reference
- [Ashok IT Theorethical CI/ CD Youtube Video](https://youtu.be/Ri-URt8gPCk)
- [Ashok IT Practical CI/ CD Youtube Video](https://www.youtube.com/watch?v=4cG7dWKbrC8)
- [Ashok IT DevOps Documentation](https://github.com/ashokitschool/DevOps-Documents)
- [Ashok IT Backup and Restore](https://www.youtube.com/watch?v=5Tb-AOUFuKQ&t=106s)
- [Setting up agent](https://dev.to/faruq2991/setting-up-jenkins-ssh-build-agents-a-complete-troubleshooting-guide-3pl#:~:text=Jenkins%20Controller:%20A%20running%20Jenkins,Scope:%20Global)
