# Linux_Server_Configuration

## Overview
Configuring a linux server to host a web app securely

## Server Details
IP address: 54.166.229.106  
SSH port: 2200  
URL: [Link](http://ec2-54-166-229-106.compute-1.amazonaws.com)

## Setup
* Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/):
  * Log in to lightsail.If you don't already have an account please create one.
  * Create a new instance
  * Choose a Ubuntu instance image
  * Choose your instance plan($5 month - first month free for this project)
  * Choose a hostname for your instance
  * Once it starts running user will get a public IP address
  * Download the default key pair ad copy to /.ssh folder
  * On terminal enter `chmod 600 ~/.ssh/key.pem`
  * SSH into your instance by `ssh -i ~/.ssh/key.pem ubuntu@54.166.229.106`
* Create a new user named 'grader':
  * `sudo adduser grader`
  * install finger to check if user has been added `sudo apt-get install finger`(optional)
  * `finger grader`
* Give grader the permission to sudo:
  * Create a new file under the sudoers directory `sudo nano /etc/sudoers.d/grader`.Edit this file with the following text `grader ALL=(ALL:ALL) ALL` and save the file
* Update and upgrade all currently installed packages:
  * Find updates: `sudo apt-get update`
  * Upgrade: `sudo apt-get upgrade`
* Change the ssh port from 22 to 2200 and other SSH configurations:
  * `nano /etc/ssh/sshd_config` change the port from 22 to 2200
  * Change `PermitRootLogin prohibit-password` to `PermitRootLogin no` to disallow root login
  * Restart ssh service by `sudo service ssh restart`
* Configure the Uncomplicated Firewall(UFW) to allow only incoming connections for SSH(port 2200), HTTP(port 80) and NTP(port 123)
  * Check ufw status to make sure it's inactive `sudo ufw status`
  * Deny all incoming by default `sudo ufw default deny incoming`
  * Allow outgoing by default `sudo ufw default allow outgoing`
  * Allow SSH on port 2200 `sudo ufw allow 2200/tcp`
  * Allow HTTP on port 80 `sudo ufw allow 80/tcp`
  * Allow NTP on port 123 `sudo ufw allow 123`
  * Turn on the firewall `sudo ufw enable`
* Configure key based authentication for user 'grader':
  * On the local machine generate SSH key pair using `ssh-keygen`
  * On another terminal login to grader account using password set during user creation `ssh -v grader@54.166.229.106 -p 2200`
  * Create .ssh directory `mkdir .ssh`
  * Create file to store key `touch /.ssh/authorized_keys`
  * Copy contents of generated key 'graderkey' from local machine in the file you just created 
  * Set permission for the file `chmod 700 .ssh ` `chmod 644 .ssh/authorized_keys`
  * Change `PasswordAuthentication` to no `nano /etc/ssh/sshd_config`
  * login with key pair `ssh grader@54.166.229.106 -p 2200 -i ~/.ssh/graderkey`
* Configure local timezone to UTC `sudo dpkg-reconfigure tzdata`
* Install and configure Apache to serve a Python mod_wsgi application
  * Install Apache `sudo apt-get install apache2`. Confirm Apache is working by visiting your public IP address
  * Install mod_wsgi `sudo apt-get install libapache2-mod-wsgi python-dev`
  * Verify wsgi is enabled `sudo a2enmod wsgi`
  * Restart Apache `sudo apache2ctl restart`
* Install git `sudo apt-get install git`
* Create Flask app (source: [Digitalocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
  * `cd /var/www`
  * `sudo mkdir catalog`
  * `cd catalog`
  * `sudo mkdir catalog`
  * `cd catalog`
  * `sudo mkdir static templates`
  * `sudo nano __init.py__`
     ```python
     from flask import Flask
     app = Flask(__name__)
     @app.route("/")
     def hello():
         return "Hello, world (Testing!)"
     if __name__ == "__main__":
        app.run()
     ```
* Install Flask
  * `sudo apt-get install python-pip`
  * `sudo pip install virtualenv`
  * `sudo virtualenv venv`
  * `sudo chmod -R 777 venv`
  * `source venv/bin/activate`
  * `pip install flask`
  * `sudo python __init.py__`
  * It should display " Running on http://localhost:5000/" or "Running on http://127.0.0.1:5000/ which means the app has been configured properly.
  * To deactivate the environmaent `deactivate`
* Configure and enable new virtual host:
  * Create host config file `sudo nano /etc/apache2/sites-available/catalog.conf`
  * Add the following lines of code to the file:
   ```python
    <VirtualHost *:80>
    ServerName 34.201.114.178
    ServerAdmin admin@34.201.114.178
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
      Order allow,deny
      Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
      Order allow,deny
      Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
   ```
  * Save and close the file
  * Enable virtual host `sudo a2ensite catalog`
* Create wsgi file
  * `cd /var/www/catalog`
  * `sudo nano catalog.wsgi`
  ```python
  #!/usr/bin/python
  import sys
  import logging
  
  activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py`
  execfile(activate_this, dict(__file__=activate_this)
  
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'Add your secret key'
  ```
  * save file
  * `sudo service apache2 restart`
* Clone github repo:
  * `sudo gitclone https://github.com/lubgade/item_catalog`
  * Make sure to get all the hidden files `shopt -s dotglob`
  * Move files from clone directory to catalog `mv /var/www/catalog/item_catalog/* /var/www/catalog/catalog/`
  * remove clone directory `sudo rm -r item_catalog`
* Make .git inaccessible
  * from `cd /var/www/catalog/` create .htaccess file `sudo nano .htaccess`
  * paste in `RedirectMatch 404/\.git`
  * save file
* Install dependencies:
  * `source venv/bin/activate`
  * `pip install httplib2`
  * `pip install requests`
  * `sudo pip install --upgrade ouath2client`
  * `sudo pip install sqlalchemy`
  * `pip install Flask-SQLAlchemy`
  * `sudo pip install python-psycopg2`
  * `pip install PIL`
  * `pip install Image`
* Install and configure PostgreSQL:
  * Install postgres `sudo apt-get install postgresql`
  * Install additional models `sudo apt-get install postgresql-contrib`
  * Config database_setup.py `sudo nano database_setup.py`
  * `engine = create_engine('postgresql://catalog:catalog-password@localhost/jewelrydb')`
  * Repeat for project.py
  * Copy project.py file to __init__.py `sudo mv project.py __init__.py`
  * Add user catalog `sudo adduser catalog`
  * Login as postgres super user `sudo su - postgres`
  * Enter postgres `psql`
  * Create user catalog `CREATE USER catalo WITH PASSWORD 'catalog-password';`
  * Change role of user catalog to createDB `ALTER USER catalog CREATEDB;`
  * List all users to verif `\du`
  * Create new database with owner catalog `CREATE DATABASE jewelrydb WITH OWNER catalog;`
  * Connect to database `\c jewelrydb`
  * Revoke all rights `REVOKE ALL ON SCHEMA public FROM public;`
  * Give access to user catalog `GRANT ALL ON SCHEMA public TO catalog;`
  * Exit postgres `\q`
  * Logout from postgres super user `exit`
  * Make sure no remote connections to the database are allowed by checking the host based authentication file  
  `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`. It should look like this:
  ```python
   local   all             postgres                                peer
   local   all             all                                     peer
   host    all             all             127.0.0.1/32            md5
   host    all             all             ::1/128                 md5
  ```
  * Setup your database schema `python database_setup.py`
* Fix OAuth to work with hosted application:
  * Go to [link](http://www.hcidata.info/host2ip.cgi) to get your host name by entering your public IP address
  * Open Apache config file `sudo nano /etc/apache2/sites-available/catalog.conf`
  * Below `ServerAdmin` paste `ServerAlias YOURHOSTNAME`
  * In google developer console add your host name and IP address to Authorized Javascript origins and Authorized redirect URI's. 
  * Download the updated client_secrets json & update in __init__.py(your old python.py)
  * Enable virtual host `sudo a2ensite catalog`
* To upload image files for the app item_catalog:
  * Change owner of all files in the project from root to the user www-data which Apache web server uses  
  `sudo chown -R www-data:www-data /var/www/catalog`
  * Restart Apache server `sudo service apache2 restart`
  
  

  
  
 
  
