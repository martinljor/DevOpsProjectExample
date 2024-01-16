# DevOps Project Example

Welcome to my GitHub repository, designed with the purpose of providing a comprehensive and detailed guide to systematically address the various roles that can be encountered in a DevOps environment. This repository aims to offer a step-by-step approach, highlighting best practices and strategies applicable to DevOps projects, with the goal of contributing to the development of specialized skills in this discipline crucial for the efficient implementation of technological solutions.

# Summary

Based on Amazon Web Services (AWS), I am going to use the following sequence:

![SEquence2](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/SEquence2.png)



| Content                                       |
| ----------------------------------------------- |
| [Create Jenkins](#create-jenkins)                |
| [Create Jenkins Agent](#1-create-jenkins-agent)|
| [Install Docker](#2-install-docker)         |
| [Configure SSH for Jenkins](#3-configure-ssh-for-jenkins) |
| [Jenkins Configuration](#4-jenkins-configuration)   |
| [Integrate Maven to Jenkins](#integrate-maven-to-jenkins)    |
| [Create SonarQube](#create-sonarqube)                      |
| [Integration between Jenkins and SonarQube](#integration-between-jenkins-and-sonarqube)  |
| [Docker - Build & Push](#docker-build--push)           |
| [Trivy Stage](#trivy-stage)                             |
| [Kubertenes + AWS EKS](#kubertenes--aws-eks)           |
| [Create ArgoCd Services](#create-argocd-services)                             |
| [Join ArgoCd <--> EKS](#join-argocd----eks)                             |
| [ArgoCd --> Gitops Repo](#trivy-stage)                             |
| [Fully automation with all the services configured.](#trivy-stage)                             |




## Create [Jenkins](https://www.jenkins.io/)

Go to the AWS console and create an EC2 instance with Ubuntu (free tier eligible):

![Create Jenkins Instance](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/15f6c045-71d1-44ec-8824-d965774b2a42.png)

- 15GB for storage also (free tier eligible).
- Public IP Enable.
- Create a key pair.

Connect to the VM using the key and ubuntu user, then run the following commands:

```bash
sudo apt update
sudo apt upgrade
```

Change the hostname of the VM using the file `/etc/hostname` and then reboot:

```bash
sudo nano /etc/hostname  # I used jenk-master
```

Because Jenkins works at port 8080, it's necessary to edit the security group associated with the EC2 and add the 8080 port:

![Security Group](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/98ee1995-acbf-45dd-9a03-436182a0422f.png)
![Security Group](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/c322b69e-30b2-4adb-aceb-1ac261cd43ea.png)

When the VM comes back, install Java:

```bash
sudo apt install openjdk-17-jre
```

Check Java version with:

```bash
java -version
```

![Java Version](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/8781e177-6474-426a-9814-18112cb2abc1.png)

To install Jenkins, it's possible to use the [official Jenkins page](https://www.jenkins.io/doc/book/installing/linux/#weekly-release). Copy and paste the code:

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

### 1. Create Jenkins Agent

Go to the AWS console and create another EC2 instance with Ubuntu (free tier eligible) and the same configuration. Use the same key pair:

![Create Jenkins Agent Instance](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/bce8bf93-2e46-4f80-a4c5-04a2f1e94c07.png)

Copy the public IP and connect to the VM. Then run update and upgrade commands:

```bash
sudo apt update && sudo apt upgrade
```

Change the hostname of the VM using the file `/etc/hostname` and then reboot:

```bash
sudo nano /etc/hostname  # I used jenk-agent
```

Install Java:

```bash
sudo apt install openjdk-17-jre -y
```

![Java Installation](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/03b25a08-d8c1-43ef-b7fa-b95549a26d5b.png)

### 2. Install Docker

In the VM jenk-agent install Docker:

```bash
sudo apt-get install docker.io -y
```

Add your user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

### 3. Configure SSH for Jenkins

Add names and IPs of both servers in the file `/etc/hosts`.

Uncomment the following lines on each server in the file: `/etc/ssh/sshd_config`.

```bash
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
```

Reload the service on each VM:

```bash
sudo systemctl reload sshd
```

On each server run the command:

```bash
ssh-keygen
```

On the master VM, copy the public key:

```bash
cat .ssh/id_rsa.pub
```

Paste it on the agent VM into the file `.ssh/authorized_keys`.

Using the public IP of the Master VM and the port 8080, check the default password and enter Jenkins:

![Jenkins Login](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/abe8cd7a-6668-4d37-9735-58a4a7e3f25f.png)

After copying and pasting the password, select the option 'Install suggested plugins':

![Install Plugins](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/250c0a85-d1ab-480b-87eb-6f33f9821fb1.png)

Follow the wizard, creating the user and assigning the IP address. Finally, Jenkins is installed:

![Jenkins Installed](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/e8e3ce70-87b8-443a-9c77-57124daeaef8.png)

### 4. Jenkins Configuration.

Change in the default node the parameter 'Number of Executors' from 2 to 0.

Create a new node as 'jenk-agent':

- Mark the option 'Permanent Agent'.
- Any description.
- Number of Executors is 2.
- Remote root directory: /home/ubuntu.
- Usage: leave it by default.
- Launch method: via SSH.
- Host: Private IP address of jenk-agent.
- Credentials: Add a new one using the option 'SSH username with private key'.
  - Username is ubuntu.
  - The private key comes from the master VM: `.ssh/id_rsa`.
- Host Key Verification Strategy: Non-verifying verification strategy.

A new node has to be created:

![Jenkins Node](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/a06560ba-0f68-4a9c-b55f-55ba542a1e8d.png)

### 5. Test Jenkins communication.

Test if there is connectivity between master and agent by building a test pipeline using the default template as 'Hello World'.

![Jenkins Pipeline](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/9953c460-993b-4510-98a8-7454bca7039e.png)

![Jenkins Pipeline](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/777df04c-8c9c-4491-8f8b-e111e830bdc8.png)

Build now and wait for the results. For a successful connection, you will see:

![Jenkins Build](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/8ff4df55-8478-4989-89c9-44e857335848.png)

Delete the test pipeline.

## Integrate Maven to Jenkins

To integrate Maven its possible to use Jenkins plugins: 

Login with admin user.
Manager Jenkins --> Plugins --> Available plugins
Search: "maven" and select the following items:
Also search for "eclipse":

![Maven-eclip search](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/Maven-eclipsearch.png)

Then click "Install" and wait to finish with all items in "Success" state.

Now there is a new item in "Manage Jenkins" --> "Tools" --> "Maven installations".
Add new maven (Remember the name):

![AddNewMaven](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/NewMaven.png)

Click "Apply" and then "Save".

### Add JDK installation

Go to "Manage Jenkins" --> "Tools" --> "JDK installations" --> "Add JDK":

* Name: remember the name that you enter.
* Check the box for "Install automatically".
  * Select the option "Install from adoptium.net".
  * Select the jdk version (for this case will be v17.x.x).

![AddJDK17](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/AddJDK17.png)

Click "Apply" and then "Save".

## Add GitHub Cred to Jenkins

Go to "Manage Jenkins" --> "Credentials"
Into the "Stores scped to Jenkins" and inside the "System" store created the is an action to add more credentials:

![StoreAddCred](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/StoreAddCred.png)

* Kind: username and password.
* Scope: Global.
* Username: github username.
* Password: github password.
* ID: unique name for identification.

![GitHubCreds](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/GitHubCreds.png)


## Create pipeline script = Jenkinsfile for Build

Go to your repository and create a new file "Jenkinsfile" with the following content:

```bash
pipeline {
    agent { label 'Jenk-Agent'}
    tools {
        jdk 'Java17'
        maven 'MyMaven'
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'GitHub-martinljor', url: 'https://github.com/martinljor/DevOpsProjectExample-Testfile'
            }
        }
        stage ("Build Application"){
            steps {
                sh "mvn clean package"
            }
        }   
    
        stage ("Test Application"){
            steps {
                sh "mvn test"
            }
        }
    }
}
```

## Test Artifacts and CI Job

Go to Jenkins portal and create a new item:
  * Name: DevOpsProjectExample-Testfile
  * Type: pipeline

    * Check "Discard old builds"
    * "Max # of builds to keep": "2"
  
  * Pipeline
    * Definition: "Pipeline script from SCM"
    * SCM: "Git"

      * Repositories
        * Repository URL: https://github.com/martinljor/DevOpsProjectExample-Testfile
        * Credentials: Select the one that was created before.
    
    * Branck Specifier: "*/main"

"Apply" and then click "Save":

![TestfileCreated](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/TestfileCreated.png)

Next step is to test it running the option "Build Now".

![BuildSuccess](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/BuildSuccess.png)    


## Create [SonarQube](https://www.sonarsource.com/products/sonarqube/)

### AWS EC2 instance.
For SonarQube its necessary to create another AWS EC2 instance with t3.medium instance type:

  * Name: SonarQube
  * OS: Ubuntu 22.04 (free tier)
  * instance type: t3.medium 
  * Key pair: used before.
  * Storage: 15 GiB.

Click "Launch instance".

So, now there are three instances created:

![ThreeInstances](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/ThreeInstances.png)

Connect to the VM using the key and ubuntu user, then run the following commands:

```bash
sudo apt update
sudo apt upgrade
```

### [PostgreSQL](https://www.postgresql.org/)

Add PostgreSQL repository:

```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
```

Lets install PostgrSQL:

```bash
sudo apt-get -y install postgresql postgresql-contrib
sudo systemctl enable postgresql
```

Now login and create the database that SonarQube its going to use:
```bash
sudo passwd postgres
su - postgres
createuser sonar
psql 
ALTER USER sonar WITH ENCRYPTED password 'sonar';
CREATE DATABASE sonarqube OWNER sonar;
grant all privileges on DATABASE sonarqube to sonar;
\q
exit
clear
```

Because Adoptium is part of this its necessary to add their repository and install temurin-java:

```bash
sudo su
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list

apt-get update
apt-get install temurin-17-jdk -y
update-alternatives --config java
/usr/bin/java --version
exit
clear
```
### Change kernel limits & paramenters

Change the kernel limits for sonarqube user with the following command:

```bash
sudo su
cat <<EOF >>/etc/security/limits.conf
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
EOF
exit
```

Change the kernel parameter to increate mapped memory regions, finally reboot the VM:

```bash
sudo su
cat <<EOF >>/etc/sysctl.conf
vm.max_map_count = 262144
EOF

exit
sudo reboot
```

### Add inbound rule for SonarQube

Go to AWS console, select the EC2 instance of SonarQube, security tab, select default security group.

"Edit inbound rule" --> "Add rule":

 * Source: Anywhere
 * Port: 9000
 * Protocol: TCP

Click "Save rules"

### Installation of [SonarQube](https://www.sonarsource.com/products/sonarqube/)

```bash
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
sudo apt-get install unzip -y
sudo unzip sonarqube-9.9.0.65466.zip -d /opt
sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
```

Lets create an user for SonarQube and set permissions:

```bash
sudo groupadd sonar
sudo useradd -c "SonarQube User" -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R
```

#### SonarQube configuration

```bash
sudo sed -i 's/#sonar.jdbc.username=/sonar.jdbc.username=sonar/g' /opt/sonarqube/conf/sonar.properties

sudo sed -i 's/#sonar.jdbc.password=/sonar.jdbc.password=sonar/g' /opt/sonarqube/conf/sonar.properties

echo "sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube" | sudo tee -a "/opt/sonarqube/conf/sonar.properties"


#Sonar service configuration

sudo su
touch /etc/systemd/system/sonar.service
cat <<EOF >>/etc/systemd/system/sonar.service
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
EOF
exit
```

#### Start and enable SonarQube services

```bash
sudo systemctl enable sonar
sudo systemctl start sonar
sudo systemctl status sonar
```

![SonarQubeStatus](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/SonarQubeStatus.png)

Using the Public IP the login portal is available:

![SonarQubeportal](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/SonarQubeportal.png)

Login with admin/admin and change the password!


## Integration between Jenkins and SonarQube

Go to SonarQube portal and go to "My account".
In the "security" tab shows a list of tokens that can be generated:

* Name: Jenk-Sonar
* Type: "Global Analysis Token"
* Expires in: "No expiration"

Click on "Generate" 

![SonarTokenCreated](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/SonarTokenCreated.png)

Then copy the token created to be use in Jenkins.

Go to Jenkins portal using the Public IP of Jenkins VM master and port 8080.

"Login" --> "Manage Jenkins" --> "Credentials" --> "Add new credentials":

* Kind: secret text
* Scope: "Global"
* Secret: paste the token
* ID: Jenk-Sonar
* Description: "Jenkins + Sonar = token"

![Jenkins-Credentials-Sonar](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/Jenkins-Credentials-Sonar.png)

### Install plugins on Jenkins

Login with admin user.
Manager Jenkins --> Plugins --> Available plugins
Search: "sonar" and select the following items:

* SonarQube Scanner
* Sonar Quality Gates
* Quality Gates

![SonarPlugin](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/SonarPlugin.png)

Click "Install"
Check the box to makes Jenkins restart when installation is complete.

#### Configuration for Jenkins and SonarQube

Manage Jenkins --> System --> SonarQube Servers --> add

* Name: SonarQubeServer
* Server URL: Private IP Address from AWS console.
* Server authentication token: Jenk-Sonar crated before.

![AddSonarQubeServer](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/AddSonarQubeServer.png)

Click "Apply" and then "Save".

#### SonarQube Scanner

For installation go to:
Manage Jenkins --> Tools --> SonarQube Scanner installations:

Click on "Add SonarQube Scanner":

* Name: SonarQube-Scanner
* Check "Install automatically"

Click "Apply" and "Save".

#### Add Stage on Jenkins pipeline file

Go to Jenkinsfile and add "SonarQube Analysis" after "Test Application" stage:

```bash
stage ("SonarQube Analysis"){
            steps {
                scripts {
                    withSonarQubeEnv(credentialsId: 'Jenk-Sonar') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
```

##### Test on Jenkins - SonarQube Analysis

Go to Jenkins portal, select the pipeline created before and select "Build Now":

![SuccessTestSonarQube](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/SuccessTestSonarQube.png)

Notification at SonarQube porta:

![SonarQubeNotportal](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/SonarQubeNotportal.png)


#### Create Webhooks in SonarQube

Go to SonarQube portal and create a Webhooks:

Login --> Administration --> Configuration --> Webhooks --> Create

* Name: Sonar-Webhook
* URL: http://PrivateIPJenkMasterVM:8080/sonarqube-webhook
* Secreate: empty

Click "Create".

Then go to Jenkinsfile and add "SonarQube Quality Gate" after "SonarQube Analysis" stage:

```bash
stage("SonarQube Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Jenk-Sonar'
                }	
            }
        }
```

##### Test on Jenkins - SonarQube Quality Gate

Go to Jenkins portal, select the pipeline created before and select "Build Now":

![SonarQubeQualGateSuccess](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/SonarQubeQualGateSuccess.png)


## Docker - Build & Push

The objective of this point is to show how to build and push docker images using pipeline script from Jenkins. You must have an account on [docker hub](https://hub.docker.com/) and create a new token at the security tab on docker page.

Login with admin user.
Manager Jenkins --> Plugins --> Available plugins
Search: "docker" and select the following items:

* Docker
* Docker Commons
* Docker Pipeline
* Docker API
* docker-build-step
* CloudBees Docker Build and Push

![DockerInstallPlugin](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/DockerInstallPlugin.png)

Click "Install"
Check the box to makes Jenkins restart when installation is complete.

Login again with admin user and go to "Manage Jenkins" --> "Credentials"
Into the "Stores scped to Jenkins" and inside the "System" store created the is an action to add more credentials:

* Kind: username and password.
* Scope: Global.
* Username: Your username of Docker Hub.
* Password: Docker Hub token created.
* ID: unique name for identification.

### Docker - Jenkinsfile env variables

Edit the jenkinsfile and add the following lines after tools in pipeline:

```bash
 environment {
	    APP_NAME = "devopsprojectexample-testfile"
            RELEASE = "1.0.0"
            DOCKER_USER = "username" # username of docker hub.
            DOCKER_PASS = 'dockerhubID' # use the same ID that was created before.
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
```

Add new stage for Docker build and push into the jenkinsfile after the stage "SonarQube Quality Gate":

```bash
stage("B&P Docker-Image") {
            steps {
              script {
                docker.withRegistry('',DOCKER_PASS) {
                  docker_image = docker.build "${IMAGE_NAME}"
                  }

                  docker.withRegistry('',DOCKER_PASS) {
                    docker_image.push("${IMAGE_TAG}")
                    docker_image.push('latest')
                    }
                }
              }
            }
           
```

Go to Jenkins portal.
Next step is to test it running the option "Build Now". The objective is to see that there is a new repository at Docker Hub and a new stage at Jenkins pipeline:

![JenkinsDockerStage](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/JenkinsDockerStage.png)

![NewRepoDockerHub](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/NewRepoDockerHub.png)

### Trivy Stage

Add the following lines to Jenkinsfile after Docker stage:

Change the parameter: dockeruser/applicationame with your data.

```bash
stage("Trivy-Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image dockeruser/applicationame:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }
 stage ('Clean artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }
```

Go to Jenkins portal. Next step is to test it running the option "Build Now".

![TrivyStageSuccess](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/TrivyStageSuccess.png)


## Kubertenes + AWS EKS

Setup a new instance in AWS EC2 with ubuntu:
Go to the AWS console and create an EC2 instance with Ubuntu (free tier eligible):

- type t2.micro
- 8GB for storage also (free tier eligible).
- Public IP Enable.
- Use the existing key pair.

Connect to the VM using the key and ubuntu user, then run the following commands:

```bash
sudo apt update
sudo apt upgrade
```

Change the hostname of the VM using the file `/etc/hostname` and then reboot:

```bash
sudo nano /etc/hostname  # I used eks-bootstrap-srv
sudo reboot
```
### Install AWSCLI

Login again and install [awscli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html):

```bash
sudo su
apt-get install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscli.zip"
unzip awscli.zip
cd aws/
./install
/usr/local/bin/aws --version
```

![awscliInstall](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/awscliInstall.png)


### Install kubectl

To install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) tool run the following commands:

```bash
sudo su
cd
curl -LO https://dl.k8s.io/release/v1.29.0/bin/linux/amd64/kubectl
chmod +x kubectl 
mv kubectl /bin
kubectl version --output=yaml
```

![kubectlInstall](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/kubectlInstall.png)

### Install eksctl

To install [eksctl](https://eksctl.io/installation/) tool run the following commands:

```bash
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

tar -zxvf eksctl_Linux_amd64.tar.gz
sudo mv eksctl /usr/local/bin/
```

![eksctlInstall](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/eksctlInstall.png)


### AWS New Role for EKS Server

Go to AWS console, search for IAM --> IAM --> Roles --> Create Role:

* AWS service
* Use case: EC2
* Permissions policies: AdministratorAccess
* Role Name: EKS-Role

Click on "Create role"

At the AWS console, go to EC2 dashboard and select the last VM created (EKS): Actions --> Security --> Modify IAM role:

* Select the new role created.

Click on "Update IAM role".

### Create EKS Cluster using eksctl

From the eks-bootstrap-srv its time to create the eks cluster using the following command:

```bash
eksctl create cluster --name DevOpsExampleEKS-cluster \
--region us-east-1 \
--node-type t2.small \
--nodes 3 
```

AWS Cloudformation in progress (its take some time to finish):
![CloudFormationEKS](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/CloudFormationEKS.png)

![EKSCommandCreation](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/EKSCommandCreation.png)

Wait until its finished.

## Create [ArgoCd](https://argoproj.github.io/cd/) Services

In this point we will be able to deploy ArgoCd in an EKS cluster and use it for Continuous Deployment process.

Official references: 
```bash
https://argo-cd.readthedocs.io/en/stable/getting_started/?_gl=1*11susc2*_ga*MjAyNTcwMDk0MS4xNzA1MzI5OTMx*_ga_5Z1VTPDL73*MTcwNTMyOTkzMC4xLjAuMTcwNTMyOTk0OC4wLjAuMA..
```

Its necessary to have an eks cluster running for this step:

```bash
kubectl get nodes
```

![GetNodes](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/GetNodes.png)

Create namespace for ArgoCd and apply default configuration:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl get pods -n argocd
```

![GetNodesArgoCd](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/GetNodesArgoCd.png)

For API server interaction you need to deploy the CLI:
```bash
curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.9.3/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```

Change the argocd-server service type to LoadBalancer:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

### Wait a couple of minutes

 kubectl get svc -n argocd
```

#### ArgoCd admin password

The password is autogenerated and stored in the secret called argocd-initial-admin-secret in the argocd installation namespace.

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```

Took the password output and decode:

```bash
echo "password" | base64 --decode
```

![ArgoCdpassword](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/ArgoCdpassword.png)



### Access to ArgoCd portal

Using the External IP that showed the command:

```bash
kubectl get svc -n argocd
```

Copy and paste it in a browser and login with the admin password desovered.

Go to user info --> update password --> change the password!

### Join ArgoCd <--> EKS

From EKS bootstrap server run the following command:

```bash
argocd login externalIP.amazonaws.com --username admin
```

Now from ArgoCd console check the connection between both services:

Login with admin --> Settings --> Clusters:

![EKSClusterArgoCD](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/EKSClusterArgoCD.png)


Also add the EKS cluster with ArgoCd command line:

```bash
sudo su
kubectl config get-contexts
### take the name and paste in the next command

argocd cluster add NAME-EXPOSED --name DevOps-eks-cluster
```

Look again at ArgoCd Cluster UI:

![ArgoCdEKSAdd2](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/ArgoCdEKSAdd2.png)



## ArgoCd --> Gitops Repo

At the ArgoCd portal its necessary to add a GitHub repository:

Login to ArgoCd Admin portal --> Settings --> Repositories --> Connect Repo.

* Via HTTPS
* Type: git
* Project: "DevOpsProject"
* Repositorty URL
* Username: (Github username)
* Password: (token created)

![GitOpsConnectionSuccess](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/GitOpsConnectionSuccess.png)


Now create a new App from the ArgoCd console:

Applications --> New App

* Application Name: devopsprojectexample-testfile
* Project Name: default
* Sync policy: Automatic
  * check "Prune resources" & "Self heal"

* Repository URL: Select the one was created.
* Path: ./
* Destination: Select the EKS cluster connected.
* Namespace: default

Click on "Create"

Wait to finish with the sync and you should see all green:

![ArgoCdAllSynced](https://github.com/martinljor/DevOpsProjectExample/blob/main/images/ArgoCdAllSynced.png)

### GitOps Repo

At this repo i created 2 files:

* service.yaml
* deployment.yaml

#### service.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: devopsprojectexample-service
  labels:
    app: devopsprojectexample-testfile
spec:
  selector:
    app: devopsprojectexample-testfile

  ports:
    - port: 8080
      targetPort: 8080

  type: LoadBalancer
```

#### deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: DevOpsProjectExample-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devopsprojectexample-testfile
  template:
    metadata:
      labels:
        app: devopsprojectexample-testfile
    spec:
      containers:
        - name: devopsprojectexample-app
          image: thegangg/devopsprojectexample-testfile:latest
          resources:
            limits:
              memory: "256Mi"
              cpu: "500m"
          ports:
            - containerPort: 8080
```

### Service running on pods

if the items were done correctly you can see a tomact default page running as your application.

To do that you have to copy the external IP address using the command:

```bash
kubectl get svc
```

Use the port 8080 and in the folder webapp/ you can see the application itself.


## Fully automation with all the actors

Go to Jenkins portal and add new item:

* Name: devops-example-project-cd
* Pipeline
  * Description: 
  * Check discard old builds
  * Max # of builds to keep: 2
  * Check "This project is parametrized" --> Add parameter.
    * String parameter:
      * Name: IMAGE_TAG
  * Check "Trigger builds remotely"
    * Authentication token: devops-example-project-token
  * Pipeline;
    * Definition: "Pipeline script from SCM"
        * SCM: Git
          * Repository URL: GitHub URL of the app
          * Credentials: 
        * Script path: Jenkinsfile
        * Branck Specifier: "*/main"

Click "Apply" and then "Save"

### Edit Jenkins file

Go to the repository where the Jenkisfile is and edit that file. Add the following stage after "Cleanup artifacts" stage:

```bash
stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user martin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'JenkinsMasterDNSname:8080/job/devops-example-project-cd/buildWithParameters?token=devops-example-project-token'"
                }
            }
       }
```

In the same file, look for (at begining) environment and add the followin line after IMAGE_TAG:

```bash
JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
```


### Add token name on Jenkins

Give access to the username that you insert in the previous stage. Go to Jenkins portal --> username --> configure --> add new token:

![AddnewtokenJenkins](https://github.com/martinljor/DevOpsProjectExample/blob/main/imagesAddnewtokenJenkins.png)

Click "Generate" and then copy the token.

Then go to Jenkins portal --> Manage Jenkins --> Credentials --> Add new credentials

* Kind: Secret text
* Scope: global
* Secret: Paste the token.
* ID: JENKINS_API_TOKEN

Click on "Create"


### Create pipeline for gitops repo

Go to GitHub repository where deployment and service.yaml are and add new file:

Name: Jenkinsfile
Content:

```bash
pipeline {
    agent { label "jenk-agent" }
    environment {
              APP_NAME = "devopsprojectexample-testfile"
    }

    stages {
        stage("Cleanup") {
            steps {
                cleanWs()
            }
        }

        stage("Check") {
               steps {
                   git branch: 'main', credentialsId: 'GitHub-martinljor', url: 'https://github.com/martinljor/DevOpsProjectExampleGitOps'
               }
        }

        stage("Update deployment tags") {
            steps {
                sh """
                   cat deployment.yaml
                   sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                   cat deployment.yaml
                """
            }
        }

        stage("Push to Git") {
            steps {
                sh """
                   git config --global user.name "martinljor"
                   git config --global user.email "tinchom@gmail.com"
                   git add deployment.yaml
                   git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'GitHub-martinljor', gitToolName: 'Default')]) {
                  sh "git push https://github.com/martinljor/DevOpsProjectExampleGitOps main"
                }
            }
        }
      
    }
}
```


### Add Schedule on CI

Go to Jenkins and edit the first item that was created:

* Check the option "Poll SCM" and add "* * * * *"

Click "Apply" and then "Save"

With this last parameter it will monitor the repository of the app and every change/commit executed it will run the CI/CD stages!

![CDpipeline](CDpipeline.png)



Thanks a lot if you reach at this line !! :) 
