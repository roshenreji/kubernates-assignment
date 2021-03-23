# Deploying Kubernates through Jenkins pipeline (where jenkins running as a docker container)

### Step 1: Start your jenkins container

### Step 2: Install the docker compose file inside the jenkins container


##### a) Go inside the container as root
````
docker exec -it -u root <nameOfContainer> bash
````

##### b) Give full permissions to the container

````
chmod 777 /var/run/docker.sock
````

##### c) Install Docker Compose inside it

````
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
````


### Step 3: Install kubectl and Minikube inside the jenkins  container

##### a)  Update Ubuntu dependencies

````
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https
````

##### b)  Install VirtualBox on Ubuntu

````
$ sudo apt-get install -y virtualbox virtualbox-ext-pack

````

##### c)  Install kubectl

````
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ sudo touch /etc/apt/sources.list.d/kubernetes.list 
$ echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubectl
````

##### d) Install Minikube

````
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.28.2/minikube-linux-amd64
$ chmod +x minikube && sudo mv minikube /usr/local/bin/
````


##### e) Start the minikube 
````
$ minikube start
$ kubectl api-versions
````

### Step 4. Download all the dependency for Kuberntes and Docker
### Step 5. Add the following as the Jenkinsfile
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

### Step 6. Add your Credentials of Docker Registry in manage credentials
##### a) Select Kind as "Username with password"
##### b) Give Username as the username for dockerhub registry
##### c) Give Password  
##### d) Give one Id and after saving copy that id to the pipline

### Step 7. Build the pipeline