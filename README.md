# Linux Server Configuration

Below are instructions for creating and configuring a secure Linux server for a Python Flask application that uses a PostgreSQL database. This is part of the requirements for completing the Udacity Full Stack Nanodegree program.

A server has been setup using these instructions at the destination below: 
  * **Server IP Address**: 34.220.96.123
  * **SSH Port**: 2200 
  * **User**: grader
  * **Domain**: [http://catalog.jonpiehl.com/](http://catalog.jonpiehl.com/)

## Server Setup Instructions

*These instrustions are for setting up a Linux server on AWS Lightsail with a Unix-based local machines (MacOS or Linux). The instructions can be modified to work with other linux server providers and for Windows users.* 

### Get a server

Create a Ubuntu Server on [AWS Lightsail](https://lightsail.aws.amazon.com/)

   * Click the link above and either login to an AWS account or create a new AWS account.
   * Once logged in, click the **Create instance** button.
   * Change the **AWS Region and Availability Zone** if you'd like. It doesn't mater what region or availability zone you select.
   * Select **Linux/Unix** as the Platform. Then select the **OS Only** option and choose any version of **Ubuntu**.
   * Select any plan *(the lowest plan is all that's needed for this project)*.
   * Name the instance or leave the default name *(the name of the instance isn't important)*
   * Click **Create instance**

### Get the default SSH key

Rather than using a password to login to the server, AWS requires users to login using a SSH key. This is more secure than using a password. [Learn more about SSH keys](https://wiki.archlinux.org/index.php/SSH_keys)

   * Go to the Lightsail account page *(near the top of the window click Account > Account)*
   * Select the **SSH keys** tab
   * **Download** the default private key
   * Move the downloaded file to the `~/.ssh` directory *(If a .ssh directory doesn't exisit, then create one)*
   * Rename the file to something shorter and easier to remember, like `ubuntu_flask_server` *(the name doesn't matter and remember that Linux doesn't require file extentions)*
   * Open the **Terminal** *(if it isn't already)* and enter the command `chmod 600 ~/.ssh/[FILENAME]` where `[FILENAME]` is the name given to the ssh key file. *(This changes the permission so that only the owner (you) can read and write the file. It's a security measure so that other users on your computer or network can't access the file to login to the server.)*
   * Login to the server using the SSH key by entering the following command into the terminal `ssh ubuntu@[IP ADDRESS] -i ~/.ssh/[FILENAME]`, where `[IP ADDRESS]` is the ip address of the Lighsail server instance and `[FILENAME]` is the name given to the ssh key file.
   * The first time you ssh into a remote server, you'll get a warning message. Type `yes` and press `enter`.

## Secure Server

### Update Packages

It's good practice to update the default installed packages when you login to the server for the first time. Run the following command while logged into the server:

```
sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade
```

### Change the SSH port

The default SSH port is 22. It's good practice to change the port number to block people from trying to login on the default port. We'll be changing the SSH port number to 2200.

  * Edit the SSH config file to allow logins from a different port.
  * Run the command `sudo nano /etc/ssh/sshd_config`.
  * Change the line `Port 22` to `Port 2200`.
  * Find `PermitRootLogin` and change it to `no`.
  * Save `ctrl+x` > `y` > `enter`
  * Restart the SSH service to load the changes to the config file `sudo service ssh restart`.

### Setup a firewall

A firewall makes it so that the server only allows connections via specific ports. We'll be using Uncomplicated Firewall (UFW) to setup a firewall. The following commands initially blocks all ports, then opens certain ones for SSH (2200), HTTP (80), and NTP (123).

**1. Configure UFW**
  * Ensure UFW is installed: `sudo apt-get install ufw`
  * Ensure UFW is NOT activated: `sudo ufw status`
     * This should return `Status:inactive`. If it doesn't then run `sudo ufw disable`
  * Run the following commands:
  
**WARNING: ONLY DO THIS IF YOU FOLLOWED THE "CHANGE SSH PORT" INSTRUCTIONS ABOVE. This blocks access to the server via the default SSH port. If you haven't changed the SSH port to 2200 as instructed above, then you will be locked out of the server forever.**

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw deny 22
sudo ufw enable
y
exit
```

**2. Configure AWS Firewall**
  * Goto your AWS Lightsail dashboard and open the server instance by clicking the name or clicking the three vertical dots and selecting Manage.
  * Click the **Networking** tab.
  * Click **Edit Rules** under the Firewall section.
  * Click the **x** next to port 22.
  * Click **+ Add another**, keep the application dropdown as **Custom**, change the Protocol to **UDP**, and enter **123** as the port.
  * Click **+ Add another**, keep the application dropdown as **Custom**, keep the Protocol to **TCP**, and enter **2200** as the port.
  * You should now have **three** ports opened - HTTP:TCP:80, Custom:UDP:123, Custom:TCP:2200.
  * Click **Save**

## Add user

### Create a new SSH key for new user

  * On your **local** terminal, enter the following code:
```
ssh-keygen -f ~/.ssh/grader_key
chmod 600 ~/.ssh/grader_key
```
*You can enter a passphrase for the ssh key to make it more secure, but it's not necessary. Leave blank (press enter) if you don't want to have to enter the passphrase when logging into the server*

  * View the public key by entering `cat ~/.ssh/grader_key.pub`
  * Copy the contents of the file that are displayed in the terminal.
  * This will be pasted into a file on the server in a few steps. Keep this terminal window open for reference later and open a new terminal window for the next steps.

### Create a new user named 'grader'

*The following commands create a user named `grader`, but similar steps can be followed to create another user named something else.*

  * SSH into the server with the default user: `ssh ubuntu@[IP ADDRESS] -p 2200 -i ~/.ssh/[FILENAME]`
  * Create a new user: `sudo adduser grader`
     * Enter a password twice. Leave blank if you don't want the user to have to enter a sudo password. Don't worry, they can't login to the server using a blank password (a ssh key is required).
     * Enter information about user. Optional. Leave blank if you'd like.
     * Give user **sudo** access: `sudo adduser grader sudo`

### Add public ssh key to server

  * While still logged into the server, enter `sudo nano /home/grader/.ssh/authorized_keys`
  * Paste the contents of the public ssh key from the previous section, save and exit.
  * On the server terminal, enter the following commands:
```
sudo chmod 700 /home/grader/.ssh
sudo chmod 644 /home/grader/.ssh/authorized_keys
sudo chown -R grader:grader /home/grader/.ssh
sudo service ssh restart
```
  * You are now able to ssh into the server as **grader** using the command `ssh grader@[IP ADDRESS] -p 2200 -i ~/.ssh/grader_key

### Simplify SSH Login

It can be a hassle having to remember the ip address of the server and the location of the ssh key everytime you login to the server. Thankfully, there's a way to simplify this process by editing the SSH config file.

* On your **local** terminal, run the following command `sudo nano ~/.ssh/config` and paste in the following code:  
```
Host ubuntu-lightsail
  HostName [IP ADDRESS]
  Port 2200
  User ubuntu
  IdentityFile [PATH TO SSH KEY]
  IdentitiesOnly yes

Host grader-lightsail
  HostName [IP ADDRESS]
  Port 2200
  User grader
  IdentityFile ~/.ssh/grader_key
  IdentitiesOnly yes
```
   * Press `ctrl+x`, then `y` and `enter`
   * [IP ADDRESS] = The Public IP Address of the Lightsail instance found on the Network tab
   * [PATH TO SSH KEY] = the path to the private ssh key (~/.ssh/udacity_flask_server)
   * *The **Host** can be named anything you'd like. It's best if it's easy to remember and descriptive of the server*

Now you can use the following simplifed SSH commands and the SSH config file will pass along the rest of the info.
```
ssh ubuntu@ubuntu-lightsail
```
OR
```
ssh grader@grader-lightsail
```

## Configure Server

### Change timezone to UTC

  * SSH into the server as **grader**
  * Enter the following code: `sudo dpkg-reconfigure tzdata`
  * Choose `Etc` or `None of the above` and select `UTC`

### Install packages

  * Login to the remote server as **grader** *(if you aren't already)*
  * Enter the following command: `sudo apt-get install apache2`
  * In a browser, enter the ip address of the server. If you see the default Apache2 Ubuntu page, then Apache was sucessfully installed
  * Enter the following commands in the server terminal:
```
sudo apt-get install libapache2-mod-wsgi-py3
sudo a2enmod wsgi
sudo service apache2 start
sudo apt-get install git
```

### Create project directory

  * SSh into the remote server as **grader** *(if you aren't already)*
  * Create a new directory for the project files: `sudo mkdir /var/www/catalog`
  * And create a directory for the application file: `sudo mkdir /var/www/catalog/catalog`
  * Change the owner of the directories to **grader**: `sudo chown -R grader:grader /var/www/catalog`

### Git deployment to remote server

  * SSH into the remote server as **grader** *(if you aren't already)*
  * Make a new directory to house Git repos: `sudo mkdir /var/repo`
  * Make a new directory for the project's Git repo: `sudo mkdir /var/repo/catalog.git`
  * *(the project's repo directory can be named anything you'd like)*
  * Change the owner of the repo directory (and all child directories/files) to **grader**: `sudo chown -R grader:grader /var/repo`
  * Navigate to the project's repo directory: `cd /var/repo/catalog.git`
  * Create a Git repo tree (but not the source files): `git init --bare`
  * Create a hook to run after commits are pushed to the git repo: `nano hooks/post-receive`
  * Enter the following code into the new file:
```
#!/bin/sh
git --work-tree=/var/www/catalog/catalog --git-dir=/var/repo/catalog.git checkout -f

```
  * Save & exit
  * Give everyone executable privilages to the file" `chmod +x hooks/post-receive`

## Deploy project to server

### Push git repo to remote server

  * In your **local** terminal, navigate to your project directory
  * If you haven't already, initialize git `git init`, configure `.gitignore`, stage all files `git add .` and commit `git commit -m '[COMMIT MESSAGE]`
  * Add the git repository on the server as a new remote repo: `git remote add live ssh://grader@grader-lightsail/var/repo/catalog.git`
  * ***live** is the name of the remote repo. It can be named anything you'd like, such as server or lightsail.*
  * Push repo to the server: `git push live master`
  
### Adjust code for remote web server

  * SSH into the remote server as **grader** *(if you aren't already)*
  * Navigate to the application directory: `cd /var/www/catalog/catalog
  * Rename the main app file: `mv application.py __init__.py`
  * Edit file: `nano __init__.py`
  * Near the bottom of the file, replace:
```
app.debug = True
app.run(host='0.0.0.0', port=5000)
```
  * With: `app.run()`
  * Near the top of the file, replace the entire line that includes `engine = create_engine`
  * With `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
  * Save & exit
  * Edit the database setup file: `nano database_setup.py`
  * Replace the entire line that includes `engine = create_engine`
  * With `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
  * Edit the models file: `nano models.py`
  * Replace the entire line that includes `engine = create_engine`
  * With `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
  * Create a WSGI file for processing web requests: `nano /var/www/catalog/catalog.wsgi`
  * Paste the following code into the file:
```
activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
  * Save & exit

### Install a virtual environment

A virtual environment allows different packages and package versions to be installed within the same server instance, but within different 'environments'. It's always good practice to setup a virtual environment in the event multiple applications are hosted on the same server.

  * SSH into the remote server as **grader** *(if you aren't already)*
  * Enter the following commands:
```
sudo apt-get install python3-pip
sudo pip3 install virtualenv
cd /var/www/catalog/catalog
virtualenv -p python3 venv3
sudo chown -R grader:grader venv3/
. venv3/bin/activate
```
  * The virtual environment should now be activated. You should see `(venv3)` at the front of the terminal line.

### Install package dependencies

  * SSH into the remote server as **grader** *(if you aren't already)*
  * Activate the virtual environment *(if it isn't already)* `. venv3/bin/activate`
  * Run the following commands:
```
pip install Flask
pip install httplib2
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
sudo apt-get install libpq-dev
pip install psycopg2
deactivate
```

### Virtual host

  * SSH into the remote server as **grader** *(if you aren't already)*
  * Enter the following command: `sudo nano /etc/apache2/mods-enabled/wsgi.conf`
  * Below where it says `#WSGIPythonPath directory|directory-1:directory-2:...` add the following line: `WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages`
  * Enter the following command: `sudo nano /etc/apache2/sites-available/catalog.conf`
  * Paste the following code:
```
<VirtualHost *:80>
    ServerName [IP ADDRESS]
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
  * Replace `[IP ADDRESS]` with the ip address of the Lightsail server instance.
  * Save & exit
  * Enter the following command: `sudo a2ensite catalog`
  * Enter the following command: `sudo service apache2 reload`

### Install postgreSQL

  * SSH into the remote server as **grader** *(if you aren't already)*
  * Enter the following commands:
```
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
CREATE USER catalog WITH PASSWORD 'catalog';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```
  * Edit the PostgreSQL config: `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
  * Paste the following code to the bottom of the page:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
  * Save & exit
  * Restart Apache: `sudo service apache2 restart`
  * The PostgreSQL database is now setup.
  * Activate the virtual environment and run the database setup secript:
```
. venv3/bin/activate
python /var/www/catalog/catalog/database_setup.py
deactivate
```

## SERVER IS READY TO GO

  * Enter the ip address of the Lightsail server into a browser.
  * You should see the application running.
  * If you get a 500 Internal Server Error, then check the error logs at `sudo cat /var/log/apache2/error.log`

## Special thanks

  * [Kendrick Calata](https://github.com/kcalata) - His [Linux Server Configuration](https://github.com/kcalata/Linux-Server-Configuration/blob/master/README.md) instructions provided the baseline for these instructions.
  * [Corey Schafer](https://github.com/CoreyMSchafer) - YouTube video on [deploying an application to a Linux Server](https://www.youtube.com/watch?v=Sa_kQheCnds)
  * [Kyle Robinson Young](https://github.com/shama) - YouTube video on [using Git to deploy code to a remote server](https://www.youtube.com/watch?v=9qIK8ZC9BnU).
