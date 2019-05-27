# Linux Server Configuration

Below are instructions for creating and configuring a secure Linux server for a Python Flask application that uses a PostgreSQL database. This is part of the requirements for completing the Udacity Full Stack Nanodegree program.

A server has been setup using these instructions at the destination below: 
  * **Server IP Address**: 34.220.96.123
  * **SSH Port**: 2200 
  * **Domain**: [http://catalog.jonpiehl.com/](http://catalog.jonpiehl.com/)

## Server Setup Instructions

*These instrustion are for Unix-based local machines (MacOS or Linux). Windows users can follow the instructions with slight modification.* 

### Get a Server

1. Create a Ubuntu Server on [AWS Lightsail](https://lightsail.aws.amazon.com/)
   * Click the link above and either login to an AWS account or create a new AWS account.
   * Once logged in, click the **Create instance** button.
   * Change the **AWS Region and Availability Zone** if you'd like. It doesn't mater what region or availability zone you select.
   * Select **Linux/Unix** as the Platform. Then select the **OS Only** option and choose any version of **Ubuntu**.
   * Select any plan *(the lowest plan is all that's needed for this project)*.
   * Name the instance or leave the default name *(the name of the instance isn't important)*
   * Click **Create instance**

### SSH into Server Using a SSH Key

Rather than using a password to login to the server, AWS requires users to login using a SSH key. This is more secure than using a password. [Learn more about SSH keys](https://wiki.archlinux.org/index.php/SSH_keys)

   * Go to the Lightsail account page *(near the top of the window click Account > Account)*
   * Select the **SSH keys** tab
   * Download the default private key
   * Move the downloaded file to the `~/.ssh` directory *(If a .ssh directory doesn't exisit, then create one)*
   * Rename the file to something shorter, like `ubuntu_flask_server` *(the name doesn't matter and remember that Linux doesn't require file extentions)*
   * Open the **Terminal** (if it isn't already) and enter the command `chmod 600 ~/.ssh/[filename]` *(This changes the permission so that only the owner (you) can read and write the file. It's a security measure so that other users on your computer or network can't access the file to login to the server.)*
   * In the **Terminal**, run the following command `sudo nano ~/.ssh/config` and paste in the following code:  
 ```
 Host [IP ADDRESS]
   HostName [IP ADDRESS]
   Port 22
   User ubuntu
   IdentityFile [FULL PATH TO SSH KEY]
   IdentitiesOnly yes
```
   * Press `ctrl+x`, then `y` and `enter`
   * [IP ADDRESS] = The Public IP Address of the Lightsail instance found on the Network tab
   * [FULL PATH TO SSH KEY] = the file path to the ssh key (/home/*username*/.ssh/*ssh filename*
   * *This is the config file that tells SSH what ssh key to use for which user and host. If you skip this step, then you can specify an ssh key while logging in with ssh by using the flag `-i [FULL PATH TO SSH KEY]`.*
   * Login to the server by entering the following command into the terminal `ssh ubuntu@[IP ADDRESS]`

    
