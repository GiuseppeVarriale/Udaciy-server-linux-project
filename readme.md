# Linux Server Configuration - Udacity Back-End Developer ND

## Overview  

This is the final project for Udacity's Back-end developer nanodegree.  
It consists of setting up a linux server with things like:

- SSH running on PORT 2200
- Configure UFW(Uncomplicated Firewall) Allowing only ports SSH(2200), HTTP(80) and NTP(123)
- Postgree Sql
- Apache2
- Secure the server
- Configure enforced Key-based SSH authentication
- Update all applications

## Server Detail

- Public IP address: `35.170.187.166`

- SSH port: `2200`

- The virtual private server is Amazon Lighsail.
- The Linux distribution is Ubuntu Ubuntu 18.04.2 LTS.
- The database server is PostgreSQL.
- The web application is my Item Catalog project created earlier in this Nanodegree program.

## Get a server on Lighsail

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
- Once you are login into the site, click `Create instance`. 
- Choose `Linux/Unix` platform.
- Then `OS Only`.
- Click on `Ubuntu 18.04 LTS`.
- Choose a instance plan ($5/month is ok).
- Rename your instance if you want.
- Click the `Create` button to create the instance.
- Wait for the instance to start up.

**Reference**

- Maria Surmenok, [Creating a Server with Amazon Lightsail](https://medium.com/@mariasurmenok/creating-a-server-with-amazon-lightsail-11c377cf814c)

## Setting Up the System

Using the amazon online terminal

### update the applications and packages

   >`$ sudo apt-get update`  
   >`$ sudo apt-get upgrade`  
   >`$ sudo apt-get dist-upgrade`  

- If at login the message *** System restart required *** is display, run the following command to reboot the machine:
   >`$ sudo reboot`  

### Add User grader

Add the user grader using the command:  

   >`$ sudo adduser grader`  

Make the grader a sudoer creating a file on /etc/sudoers.d/ using the comand  
   >`$ touch /etc/sudoers.d/grader`  

edit the file  
   >`$ nano /etc/sudoers.d/grader`  

make the file content be like the below content  

```console
grader ALL=(ALL) NOPASSWD:ALL
```

### Configure a key-based authentication for grade user

On your **local machine**, generate an encryption key.  
   >`$ ssh-keygen`

The system  will prompt you about the file name and location. Put grader_lightsail. If You want You can choose the name you want, but remember
to modify below in lines where i used grader_lightsail

By default, the keys will be stored in the ~/.ssh directory within your user's 
home directory.

run the command and copy the content
   >`$ cat ~/.ssh/grader_lightsail.pub`

Return to the Amazon lightsail terminal and run the comands:

```console
sudo mkdir /home/grader/.ssh
sudo nano /home/grader/.ssh/authorized_keys
```

**Paste** the copied content
save the file

run commands

```console
sudo chown -R grader:grader /home/grader/.ssh
sudo chmod 700 /home/grader/.ssh
sudo chmod 644 /home/grader/.ssh/authorized_keys
```

Can you now login as the grader user on your local SSH terminal using the command:  
   >`$ ssh grader@35.170.187.166 -p 2200 -i ~/.ssh/grader_lightsail`

### Change SSH configurations

-> changing port, disabling root login and force key-based login

Connected with grader user, run:
   >`sudo nano/etc/ssh/sshd_config`

Look for lines with tPort, PasswordAuthentication and PermitRootLogin and change to the below:

```
Port 2200
PasswordAuthentication no
PermitRootLogin no
```

Now you need to configure the Amazon Lighsail firewall like the below image:


