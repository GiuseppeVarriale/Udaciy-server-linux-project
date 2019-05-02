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

- Public IP address: ~~`35.170.187.166`~~ 

- SSH port: `2200`

- Url: Application URL: ~~[http://35.170.187.166.xip.io](http://35.170.187.166.xip.io)~~
- The virtual private server is Amazon Lighsail.
- The Linux distribution is Ubuntu Ubuntu 18.04.2 LTS.
- The database server is PostgreSQL.
- The web application is my Item Catalog project created earlier in this Nanodegree program.

## Get a server on Lightsail

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

## update the applications and packages

   >`$ sudo apt-get update`  
   >`$ sudo apt-get upgrade`  
   >`$ sudo apt-get dist-upgrade`  

- If at login the message *** System restart required *** is display, run the following command to reboot the machine:
   >`$ sudo reboot`  

## Add User grader

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

## Configure a key-based authentication for grade user

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
   >`$ ssh grader@35.170.187.166 -p 22 -i ~/.ssh/grader_lightsail`

## Change SSH configurations

-> changing port, disabling root login and force key-based login

Connected with grader user, run:
   >`sudo nano/etc/ssh/sshd_config`

Look for lines with tPort, PasswordAuthentication and PermitRootLogin and change to the below:

```
Port 2200
PasswordAuthentication no
PermitRootLogin no
```

run:
>`sudo service ssh restart`

Now you need to configure the Amazon Lighsail firewall like the below image:  

![image](https://github.com/GiuseppeVarriale/Udaciy-server-linux-project/raw/master/images/lightsail_firewall.png)

and then access the server:
   >`$ ssh grader@35.170.187.166 -p 2200 -i ~/.ssh/grader_lightsail`

## Configure Uncomplicated Firewall

```console
$ sudo ufw status
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow ntp
```

now run:  
   >`$ sudo ufw show added`

check if all configurations are ok.

to enable run:
   >`$ sudo ufw enable`

and check the status running:
   >`$ sudo ufw status`

## Set UTC TIME and install NTP

run:

```console
   $ sudo timedatectl set-timezone UTC 
   $ sudo apt-get install ntp
```

## Install unattended-upgrades to automate the update of the packages
[Ubuntu documentation on Automatic updates](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)

Install the unattended-upgrades package:
`sudo apt install unattended-upgrades`

Check the congiguration file `/etc/apt/apt.conf.d/50unattended-upgrades` to adjust it to fit your needs. It was left at the default of security updates only.

To enable automatic updates, edit `/etc/apt/apt.conf.d/20auto-upgrades` and set the appropriate apt configuration options:

```s
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```
The above configuration updates the package list, downloads, and installs available upgrades every day. The local download archive is cleaned every week.

## Install Python 2.7
```console
$ sudo apt-get install python2.7 python-pip
```

## Install PostgreSQL
   >`$ sudo apt-get install postgresql postgresql-contrib`

To ensure that remote connections to PostgreSQL are not allowed, check that the configuration 
file /etc/postgresql/9.5cd/main/pg_hba.conf only allowed connections from the local host addresses 127.0.0.1 for IPv4 and ::1 for IPv6.

Create a PostgreSQL user called catalog with:
   >`$ sudo -u postgres createuser -P catalog`

You are prompted for a password, type **catalog10**. This creates a normal user that can't create databases, roles (users).

Create an empty database called catalog with:

   >`$ sudo -u postgres createdb -O catalog catalog`

## Install Apache:

   > `sudo apt-get install apache2`

Install the libapache2-mod-wsgi package:

   >`sudo apt-get install libapache2-mod-wsgi`

Enable mod_wsgi:
   >`$ sudo a2enmod wsgi`

## Install some packages  

```
$ sudo apt-get install python-psycopg2 python-flask
$ sudo apt-get install python-sqlalchemy python-pip
$ sudo pip install oauth2client
$ sudo pip install requests
$ sudo pip install httplib2
$ sudo pip install passlib
```

## Install git

   >`sudo apt-get install git`

## Clone the repository

```
$ cd /var/www/
$ sudo mkdir Catalog
$ cd Catalog
$ sudo -u www-data git clone https://github.com/GiuseppeVarriale/ud-Catalog.git Catalog
$ sudo chown -R www-data:www-data /var/www/Catalog
```

## Make .git file inaccessable
```
$ sudo nano /var/www/Catalog/.htaccess
```

add line  
```
RedirectMatch 404 /\.git
```

referece:
[Hide Git Repos on Public Sites](https://davidegan.me/hide-git-repos-on-public-sites/)

### Modify the app structure to deploy

- First go to the directory
   >`cd /var/www/Catalog/Catalog/`

- Rename you application.py
   >`sudo mv application.py __init__.py`

- Edit database_setup
   >`sudo nano database_setup.py`

Search the line

```python
engine = create_engine('sqlite:///catalog.db',
                       connect_args={'check_same_thread': False})
```

change to

```python
engine = create_engine('postgresql://catalog:catalog10@localhost/catalog')
```

- Configure fb_client_secrets.json
   >`sudo cp fb_client_secrets.json.example fb_client_secrets.json.example`  
   >`sudo nano fb_client_secrets.json`

To authentication the app use facebook api with OAUTH. You need to configure it with you app_id and app_secret
More infomation about it [**here**](https://www.udacity.com/course/authentication-authorization-oauth--ud330)  

1. Configure the app_id and app_secret on file fb_create_secrets.json 
   this is provide ou your developer facebook when you  configure the facebook api [**Here**](https://developers.facebook.com/apps).  

```json
file: fb_create_secrets.json

{
  "web": {  
    "app_id": "YOUR_APP_ID_HERE",  
    "app_secret": "YOUR_APP_SECRET_HERE"  
  }  
```

- Edit __init__.py

Change this code (line 35) from:
```python
    app_id = json.loads(open('fb_client_secrets.json', 'r').read())[
        'web']['app_id']
```

to
```python
    with app.open_resource('fb_client_secrets.json') as f:
        app_id = json.load(f)['web']['app_id']
```

change this code (line 57)
```python
   app_id = json.loads(open('fb_client_secrets.json', 'r').read())[
        'web']['app_id']
    app_secret = json.loads(
        open('fb_client_secrets.json', 'r').read())['web']['app_secret']
```
to
```python
with app.open_resource('fb_client_secrets.json') as f:
        app_id = json.load(f)['web']['app_id']
        app_secret = json.load(f)['web']['app_secret']
```

change below:
```python
    app.run(host='0.0.0.0', port=5000)
```
to it:
```python
    app.run()
```

## Install requirements

   >`$ sudo pip install -r requirements.txt`

## Set and populate database

   >`$ sudo python database_create.py`
   >`$ sudo python lotofthingsdb.py`

# Create and update catalog.wsgi file for the installation

   >`$ sudo nano /var/www/Catalog/catalog.wsgi`
Type in the following:

```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Catalog/")

from Catalog import app as application
application.secret_key = 'super_secret_key'
```

# Configure and Enable Apache2 to serve your app

   >`$ sudo nano /etc/apache2/sites-available/Catalog.conf`

paste the content below  
```xml
<VirtualHost *:80>
                ServerName catalog.com
                WSGIDaemonProcess yourapplication user=grader group=grader thre$
                ServerAdmin admin@catalog.com

                WSGIScriptAlias / /var/www/Catalog/catalog.wsgi
                <Directory /var/www/Catalog/Catalog/>
                        Require all granted
                </Directory>
                Alias /static /var/www/Catalog/Catalog/static
                <Directory /var/www/Catalog/Catalog/static/>
                        Require all granted
                </Directory>

                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
reference:[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)

### Disable the default virtual host
   >`$sudo a2dissite 000-default.conf`

### Enable the virtual host just created
   >`$ sudo a2ensite Catalog.conf`

### set the owner again (new files)
   >`$ sudo chown -R www-data:www-data /var/www/Catalog`

### To make these changes live restart Apache2
   >`$ sudo service apache2 restart`


#Run your init.py

   >`$ python __init__.py`

#Restart Apache

   >`$ sudo service apache2 restart`

#Your app is now online!
