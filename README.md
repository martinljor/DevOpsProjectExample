# DevOps Project Example

Using AWS i am going to use the following sequence:
![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/021b390f-bfa6-4d71-98da-c5f7049883fc)


* 1st Step: Create Jenkins
  Go to AWS console and create an EC2 instance with Ubuntu (free tier eligible):
  ![image](https://github.com/martinljor/DevOpsProjectExample/assets/7529358/19044121-a780-477f-b52e-7e97a7999b28)

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

  




