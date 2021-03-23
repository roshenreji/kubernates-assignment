# Deploying Kubernates through Jenkins(locally) pipeline 

### Step 1. Download all the dependency for Kuberntes and Docker
### Step 2. Add the following as the Jenkinsfile
````
pipeline{
    agent any

    stages{

    
        stage("docker build"){
            steps{
                sh "docker-compose up -d"
            }
        }

        stage("commiting the docker images"){
            steps{
                sh "docker commit kanban-ui roshenreji/kubernetes-ui"
                sh "docker commit kanban-app roshenreji/kubernetes-app" 
            }
        }

        stage("pushing the images to docker hub"){
            steps{
                withDockerRegistry([ credentialsId: "f584bcd1-20ad-4372-a6d8-aaecfaf8096e", url: "" ]){
                    sh "docker push roshenreji/kubernetes-app"
                    sh "docker push roshenreji/kubernetes-ui"
                }
            }
        }

        stage("kubernates"){
            steps{
                sh "kubectl apply -f kubernetes"
                //sh "minikube service ui-service"
            }
        }
    }
}
````

### Step 3. Add your Credentials of Docker Registry in manage credentials
##### a) Select Kind as "Username with password"
##### b) Give Username as the username for dockerhub registry
##### c) Give Password  
##### d) Give one Id and after saving copy that id to the pipline

### Step 3. Build the pipeline