# DevOps Project Example

Using AWS i am going to use the following sequence:
![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/021b390f-bfa6-4d71-98da-c5f7049883fc)


* 1st Step: Create Jenkins
  Go to AWS console and create an EC2 instance with Ubuntu (free tier eligible):
  ![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/15f6c045-71d1-44ec-8824-d965774b2a42)

    * 15GB for storage also (free tier eligible).
    * Public IP Enable.
    * Create a key pair.

Connect to the VM using the key and ubuntu user, then run sudo apt update & sudo apt upgrade
Change the hostname of the VM using the file /etc/hostname and then reboot:  sudo nano /etc/hostname # i used jenk-master

Because Jenkins works at port 8080 its necessary to edit the security group associate with the EC2 and add the 8080 port:
![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/98ee1995-acbf-45dd-9a03-436182a0422f)
![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/c322b69e-30b2-4adb-aceb-1ac261cd43ea)

When the VM comes back, install Java: sudo apt install openjdk-17-jre
Check java version with: java -version
![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/8781e177-6474-426a-9814-18112cb2abc1)

To install Jenkins its possible to use the official page of Jenkins: https://www.jenkins.io/doc/book/installing/linux/#weekly-release

Copy and paste the code:
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
![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/ab33782f-101c-4aed-b6f7-bbd6ed745826)

* 2nd step: Create Jenkin Agent
  Go to AWS console and create again another EC2 instance with Ubuntu (free tier eligible) and same configuration. Use the same key pair.
![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/bce8bf93-2e46-4f80-a4c5-04a2f1e94c07)

Copy the public IP and connect to the VM. Then run update and upgrade commands:
sudo apt update & sudo apt upgrade

Change the hostname of the VM using the file /etc/hostname and then reboot:  sudo nano /etc/hostname # i used jenk-agent
Install Java: sudo apt install openjdk-17-jre -y
![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/03b25a08-d8c1-43ef-b7fa-b95549a26d5b)

* 3rd step: Install docker
  In the VM jenk-agent install docker: sudo apt-get install docker.io -y

  Add your user to the docker group:
  sudo usermod -aG docker $USER

* 4rd step: 
  Add names and IPs of both servers in the file /etc/hosts
  Uncomment the following lines on each server in the file: /etc/ssh/sshd_config:

  PubkeyAuthentication yes
  AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2

  Reload the service on each VM: sudo systemctl reload sshd
  On each server run the command: ssh-keygen

  On master VM copy the public key: cat .ssh/id_rsa.pub
  Paste it on the agent VM into the file .ssh/authorized_keys

  Using the public IP of Master VM and the port 8080.
  Check the default password and enter to Jenkins:
  ![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/abe8cd7a-6668-4d37-9735-58a4a7e3f25f)

  After copy and paste the password select the option "Install suggested plugins"
  ![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/250c0a85-d1ab-480b-87eb-6f33f9821fb1)

  Follow the wizard creating the user and assigning the IP address. Finally Jenkins is installed:
  ![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/e8e3ce70-87b8-443a-9c77-57124daeaef8)

  Jenkin configs

  Change in the default node the parameter "number of executors" 2 to 0.
  Create a new node as jenk-agent:
    * mark the option "permanent agent"
    * any description
    * number of executors is 2
    * Remote root directory: /home/ubuntu
    * Usage: leave it by default.
    * Launch method: vi SSH.
    * Host: Private IP address of jenk-agent
    * Credentials:
        Add a new one using the option SSH username with private key.
        Username is ubuntu
        The private key comes from master VM: .ssh/id_rsa
    * Host Key Verification Strategy: non verifying verification strategy
  A new node has to be created:
![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/a06560ba-0f68-4a9c-b55f-55ba542a1e8d)

Test if there is connectivity between master and agent building test pipeline using default template as Hello World.
  ![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/9953c460-993b-4510-98a8-7454bca7039e)
 
  ![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/777df04c-8c9c-4491-8f8b-e111e830bdc8)

  Buid now and wait the results. For success connection you will see:
  ![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/8ff4df55-8478-4989-89c9-44e857335848)

  



      
  


  

  
  
  
  





  




