# Project No.3: Linux Server Configuration 
## Lenore Alford, April 2019
## Project Description

> Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The application meant to be deployed is the **ServiceApp**, previously developed for Project 2.

## Basic Info:

IP address: 18.236.129.237.

Accessible SSH port: 2200.

Application URL: 18.236.129.237.xip.io

Procedures for the completion of this project:

# Configure Linux Server

## Listing packages  
- View existing installed packages on server via the `/etc/apt/sources.list` command
  
## Update packages
- Type command `sudo apt-get update` to see the packages on server that need an update

## Upgrade packages
- Type command `sudo apt-get upgrade` to update all packages on server

## Auto remove packages that are not required
- Type command `sudo apt-get autoremove` to remove packages that are no longer required

## Install the **finger** package
- Type command `sudo apt-get install finger` to install the **finger** package 

## Create a new user called "grader"
- Type command `sudo adduser grader` to create a new user **grader**
- Type command `finger grader` to view this user

## Give "grader" sudo access
To give the user **grader** access to **sudo**, we need to add **grader** to the /etc/sudoers.d directory.

+ Type command `sudo touch /etc/sudoers.d/grader` to create a blank grader file in the /etc/sudoers.d directory

+ Type command `sudo nano /etc/sudoers.d/grader` and

add this line into the sudoers.d file: `grader ALL=(ALL) NOPASSWD:ALL`

## Give "grader" ability to log into server via SSH key pair on default port

### Local Computer

+ generate a secure SSH key-pair via the `ssh-keygen` command 

### Linux Server

+ as the ubuntu user on the Linux server, login as grader with `su grader`.

+ create a .ssh directory grader's home directory, /home/grader

+ chmod 700 /home/grader/.ssh directory

+ chmod 600 /home/grader.ssh/grader_creds file 
  Please note: I used 600 as recommended by this post on Stack Overflow: 
  https://stackoverflow.com/questions/9270734/ssh-permissions-are-too-open-error


### Local Machine

+ use following command to SSH into the Linux server as user **grader** `ssh -i ~/.ssh/<private_key> grader@18.236.129.237`


# Force Key-based Authentication; Disable Password Login

To enforce key-based authentication and disable password logins,

+ edit the /etc/ssh/sshd_config file in the Linux server with the command `sudo nano /etc/ssh/sshd_config` and check that `PasswordAuthentication no` exists.

+ disable **root** login to the server, change "prohibit password" to "no" in the PermitRootLogin line.
  

# Configure Firewall on Linux server

 + Use the following commands:

`    sudo ufw default deny incoming` 
    `sudo ufw default allow outgoing`

 + Allow the ssh on non-default port of 2200:

`sudo ufw allow 2200/tcp`

+ Allow the http on default port 80:

`sudo ufw allow www`

+ Allow the NTP on default port 123:

`sudo ufw allow ntp`

+ Enable the firewall configuration:

`sudo ufw enable`

+ Check the firewall status:

`sudo ufw status`

  
Result:
   

    To                         Action      From
--                         ------      ----
22                         DENY        Anywhere                  
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123                        ALLOW       Anywhere                  
22 (v6)                    DENY        Anywhere (v6)             
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123 (v6)                   ALLOW       Anywhere (v6)

  

# Configure firewall on Amazon Lightsail dashboard

  

+ On the Lightsail Networking tab, add a custom port 2200 as type tcp.

+ Reboot the Lightsail server.

  
# SSH to Linux server (access for grader direction)

+ Access private key from "Notes to Reviewer"

+ SSH as **grader** to the Lightsail server at port 2200:

 `ssh -i ~/.ssh/<private_key> grader@18.236.129.237 -p 2200`

# Install a Web server

## Install Apache

To install the Apache HTTP server, type the following command:

`sudo apt-get install apache2`

To test that a web server is running on port 80, type this command at your local browser or click the link:

[http://18.236.129.237/](http://18.236.129.237/)
  
# Install and configure Database Server

## Install PostgreSQL
We install PostgreSQL using this command: `sudo apt-get install postgresql`

## Create database user `catalogue`

Type the command     `sudo -u postgres psql postgres` at the terminal.  The output is as follows:

    psql (9.5.14)
    Type "help" for help.

postgres=# CREATE USER catalogue WITH PASSWORD 'password';

postgres=# ALTER USER catalogue CREATEDB;
ALTER ROLE
postgres=# CREATE DATABASE catalogue WITH OWNER catalogue;
CREATE DATABASE
postgres=# \c catalogue
You are now connected to database "catalogue" as user "postgres".
servicemenu=# REVOKE ALL ON SCHEMA public FROM public;
REVOKE
servicemenu=# GRANT ALL ON SCHEMA public TO catalogue;

Use command `psql postgresql://catalogue@localhost/catalogue` to verify that the database and user are created.

At the "catalogue=>" prompt, type `\l` to list databases and users created. This is the result (a few extra due to mishaps) :
 
                                   List of databases
    Name     |   Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-------------+-----------+----------+-------------+-------------+-----------------------
 catalogue   | catalogue | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres    | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 servicemenu | catalog   | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0   | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |           |          |             |             | postgres=CTc/postgres
 template1   | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |           |          |             |             | postgres=CTc/postgres
 
 - Type \q and then press ENTER to quit psql. 

# Configure local timezone to UTC on linux server
+ `sudo dpkg-reconfigure tzdata` on terminal and press Enter for  `None of the above` and then for `UTC`

The output of this command is as follows:

Current default time zone: 'Etc/UTC'
Local time is now:      Wed Apr 17 17:50:09 UTC 2019.
Universal Time is now:  Wed Apr 17 17:50:09 UTC 2019.
# Create Virtual Environment

+ pip install virtualenv
+ virtualenv final_env
+ activate virtual environment with `source final_env/bin/activate`

# Deploy ServiceApp Project
## Install Git
To install git at the terminal:
`sudo apt-get install git-core`

## Clone ServiceApp project
+ While in virutal environment, Go to the path `/var/www`
+ `git clone https://github.com/topplethepat/servicesApp.git servicesApp`

## Configure Item ServiceApp project for WSGI
+ To configure Apache to handle requests using the WSGI module, first install mod_wsgi:

`sudo apt-get install libapache2-mod-wsgi`

+ `sudo nano /etc/apache2/sites-available/ServiceApp.conf`

with this content: 

    `<VirtualHost *:80>
                ServerName 18.236.129.237
                ServerAdmin lenore.alford@gmail.com
                WSGIScriptAlias / /var/www/ServiceApp/serviceapp.wsgi
                <Directory /var/www/ServiceApp/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/ServiceApp/ServiceApp/static
                <Directory /var/www/ServiceApp/ServiceApp/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>`

+ Enable the ServiceApp app with this command: 
`sudo a2ensite ServiceApp`
+ Create the WSGI file: 
`sudo nano /var/www/ServiceApp/serviceapp.wsgi` 
with the content:

`
#!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/ServiceApp/")
    import ServiceApp as application
    application.secret_key = 'my_secret_key'`

+ Restart Apache: `sudo service apache2 restart`

## Install SQLAlchemy, psycopg2, oauth2client, requests
+ To install **sqlalchemy**, type the command: `sudo -c apt-get install python-sqlalchemy`
+ To install **psycopg2**, type the command: `sudo -c apt-get install python-psycopg2`
+ To install **oauth2client**, type the command: `sudo -c apt-get install python-oauth2client`
+ To install **requests**, type the command: `sudo -c apt-get install python-requests`

## Populate the database with sample data

+ Run the following commands to create and populate the **ServiceApp** database
`
python my_databasesetup.py
python populate_svcs.py
`
The database contains services children could provide for neighbors.

+ Verify that the tables exist in the **ServiceApp** database with `psql postgresql://catalogue@localhost/catalogue`. (Password is 'password') At the **psql** terminal type `\dt` and `\d` to list the tables.  

+ Verify that the tables were populated by typing simple queries at the **psql** terminal, such as `select * from User`, `select * from Service` 

# Third-party Resources

+ # How To Deploy a Flask Application on an Ubuntu VPS(https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) 
+ # stackoverflow.com
+ # http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/

Please note: per this post, I understand that since xip.io is blocked by google, the google auth part of my app will not be required: https://knowledge.udacity.com/questions/28323



 
