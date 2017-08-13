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
* Install and configure Apache to serve a Python mod__wsgi 
 
  
