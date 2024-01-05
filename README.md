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



