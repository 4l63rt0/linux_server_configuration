#Linux Server Configuration

##Project Overview

This will be a baseline installation of a Linux server hosting a web application. The server will be secure from a number of attack vectors, also a database will be configure to serve the web app in addition of the web app deployment.

Build this system will give us a depp understanding of how to build from scratch a web application, its database and how to host it

##Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

**Server IP Address:**        http://52.41.47.18

**Grader SSH:**               Attached to repository

**Grader username:password**  grader:grader

###Create Server instance at lightsail

* Create  server instance in [Amazon Lightsail](https://aws.amazon.com/ "LightSail")
  - Plataform: Linux/Unix
  - Blueprint: OS only / Ubuntu 16.04 LTS

>You might need to wait a minute to be able to connect to the terminal. After the server instance is created you will have your public IP and other information about the server

* Follow the instructions at amazon LightSail to connecto to the Terminal (SSH)


### Secure your server

* Update all currently installed packages

  <code>sudo apt-get update</code>

  <code>sudo apt-get upgrade</code>

>After the upgrade is done system will ask _What would you like to do about menu.lst?_ Go head and selet _keep the local version currently installed_

* Configure LightSail firewall to allow connections to the next ports:

  | Application | Port# |
  |-------------|-------|
  |SSH          |2200   |
  |HTTP         |80     |
  |NTP          |123    |

* Configure Uncomplicated Firewall (UFW) to only allo incoming connections from SSH (port 2200), HTTP (port 80) and NTP (port 123)


>**_Warning:_** When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.

* Deny all incoming connections

  <code>sudo ufw default deny incoming</code>

* Allow all outgoing connections

  <code>sudo ufw default allow outgoing</code>

* New port for SSH service

  <code>sudo ufw allow 2200/tcp</code>

* Allow communication to port 80

  <code>sudo ufw allow www</code>

* Allow communication to NTP service

  <code>sudo ufw allow 123/tcp</code>

* Enable UFW service

  <code>sudo ufw enable</code>

* Finally confirm the status of the service

  <code>sudo ufw status</code>

### Create User "grader"

* We will create a user with the name of "grader" and password "grader"

  <code>sudo adduser grader</code>

* Add user "grader" to the "sudo" group

  <code>sudo adduser grader sudo</code>

* To confirm that the user was installed the right way we will need to install "Finger" application. This application will be use also in the near future

  <code>sudo apt-get install finger</code>

### Create a SSH key pair for user "grader" using the ssh-keygen tool as user authentification

**This SSH key has to be generated in your local machine, no in the server**

* Create ssh-key (remember this has to be in your local computer, not in the server)

  <code>ssh-keygen</code>

* Select the directory where you are going to save your key

* (Optional) Enter a passphrase for your ssh-key (This will be like a password to use the key)

* You will get 2 files:
  - file1
  - file2.pub

>**file1** will be use for SSH remote connections

>**file2.pub** will be transfer to the server where you want to connecto

* Save the public key (file2.pub) to the server

  - Create .ssh directory at the server under "grader" account at its home directory

    <code>mkdir .ssh</code>

  - Create a new file named "authorized_keys" inside ".ssh" directory. This file will store all the public authorized_keys

    <code>touch .ssh/authorized_keys</code>

  - Go back to your local computer and copy "file2.pub" content and paste it in to the "authorized_key" file1

    <code>nano .ssh/authorized_keys</code>

  - Change permission the your "authorized_key" file

    <code>chmod 700 .ssh</code>

    <code>chmod 644 .ssh/authorized_keys</code>

  - To confirm that you were able to gain access to the server through remote connection using shh try to connect to the server from your local machine using your SSH keygen

    <code>ssh grader@52.41.47.18 -p 2200 -i ~/.ssh/file1</code>

  - If you set a passphrase you will need to enter it

  - Disable the password base login to secure the authentification process

    <code>sudo nano /etc/ssh/sshd_config</code>

    - Inside the "sshd_config" file set to "no" the "PasswordAuthentification" option

      <code>PasswordAuthentification no</code>

  - Restart "ssh" service

    <code>sudo service ssh restar</code>

### Prepapre to Deploy Project

#### Configure the local time to UTC

* Configure local time to UTC

  <code>sudo timedatectl set-timezone UTC</code>

#### Install and configure Apache to serve a Python mod_wsgi application

* Install Apache and prerequisites for mod_wsgi

  <code>sudo apt-get install apache2 apache2-utils libexpat1 ssl-cert python</code>

* To confirm that everything was installed correctly access http://localhost. You should see the default Ubuntu Apache page

  <code>curl http://localhost</code>

* Now, install mod_wsgi by running the following command

  <code>sudo apt-get install libapache2-mod-wsgi</code>

* Restart Apache service to get mod_wsgi to work

  <code>sudo /etc/init.d/apache2 restart</code>


### Install and configure PostgreSQL

* Install PostgreSQL

  <code>sudo apt-get install postgresql postgresql-contrib</code>

* Add new role named "catalog"

  <code>sudo -u postgres createuser --interactive</code>

* Create new database

  <code>sudo -u postgres createdb departmentapps</code>

* Create new user for dabatase

  <code>sudo adduser catalog</code>

* Add user to the sudo group

  <code>sudo adduser catalog sudo</code>

* Deny remote connections to database modifing the "pg_hda.conf" file.

  <code>sudo nano /etc/postgresql/9.1/main/pg_hba.conf</code>

  |Type |Database|User    |Address     |Method|
  |-----|--------|--------|------------|------|
  |local|all     |all     |            |peer  |
  |local|all     |all     |            |peer  |
  |host |all     |all     |127.0.0.1/32|md5   |
  |host |all     |all     |::1/128     |md5   |

* Grant all access to user "catalog" to database "departmentapps"

<code>sudo su -postgres</code>

<code>psql</code>

<code>GRANT ALL PRIVILEGES ON DATABASE departmentapps to catalog;</code>

<code>ALTER ROLE catalog WITH PASSWORD catalog;</code>

### Install and configure Git

* Install git

  <code>sudo apt-get install git</code>

* Configure git

  <code>git config --global user.name "John Corn"</code>

  <code>git config --global user.email "email@example.com"</code>

### Deploy the Item Catalog project

* Clone project to server

  <code>sudo git clone https://github.com/4l63rt0/UDACITY-BACK-END-PROJECT/blob/master/application.py</code>

* Enable mod_wsgi

  <code>sudo a2enmod wsgi</code>

* Create the application directory structure

  <code>cd /var/www</code>

  <code>sudo mkdir Flask</code>

  <code>cd Flask</code>

  <code>sudo mkdir Flask</code>

  <code>cd Flask</code>

* Move all your project directory content to the second Flask directory

  <code>sudo mv UDACITY-BACK-END-PROJECT/* /var/www/Flask/Flask</code>

#### Install Flask

* Install pip to install virtualenv

  <code>sudo apt-get install python-pip</code>

* Install virtualenv

  <code>sudo pip install virtualenv</code>

* Create virtual enviroment inside the Flask directory ""/var/www/Flask/Flask"

  <code>sudo virtualenv venv</code>

* Activate virtual enviroment to proceed to install Flask on it

 <code>source venv/bin/activate</code>

* Now install Flask

  <code>sudo pip install Flask</code>

* Test if installation was successful by running this command

  <code>sudo python /var/www/Flask/Flask/\_\_init__.py</code>

* Flask server should be have this output:

  <code>Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)</code>

* Deactivate virtual enviroment

  <code>deactivate</code>

#### Configure new virtualhost for Flask

* Configure FlaskApp.config

  <code>sudo nano /etc/apache2/sites-available/FlaskApp.conf</code>

* Add the following content and save it

```
<VirtualHost *:80>
        ServerName 52.41.47.18
        ServerAdmin 4l63rt0@gmail.com
        WSGIScriptAlias / /var/www/Flask/flaskapp.wsgi
        <Directory /var/www/Flask/Flask/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/Flask/Flask/static
        <Directory /var/www/Flask/Flask/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Enable virtual host

  <code>sudo a2ensite FlaskApp</code>

* Create a flaskapp.wsgi file inside the Flask directory to serve the flask app /var/www/Flask/

  <code>sudo nano /var/www/Flask/flaskapp.wsgi</code>

* Add the following content and save it

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Flask/")

from Flask import app as application
application.secret_key = 'xUldotBaq77gzfaY95BDbR6g'
```

* Restart Apache

  <code>sudo apachectl restart</code>

