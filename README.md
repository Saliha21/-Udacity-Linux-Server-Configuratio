# Udacity Linux Server Configuration:

## Project Description:

We will learn how to access, secure, and perform the initial configuration of a bare-bones Linux server. We will then learn how to install and configure a web and database server and actually host a web application.

- IP address: 3.8.126.57 

- Application URL: http://ec2-3-8-126-57.eu-west-2.compute.amazonaws.com/


## Starting:

1. Download Private Key From SSH Key on the Amazon Lightsail.
2. Move the private key file into the folder `~/.ssh` rename it lightsail_key.rsa.
3. In terminal, type: `chmod 600 ~/.ssh/lightsail_key.rsa`.
4. In terminal, type: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@3.8.126.57`


## Change the SSH port from 22 to 2200:
 - Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200, save & exit.
 - Restart SSH: `sudo service ssh restart`.

## Configure the Uncomplicated Firewall (UFW):
 Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
 1. `sudo ufw allow 2200/tcp`
 2. `sudo ufw allow 80/tcp`
 3. `sudo ufw allow 123/udp`
 4. `sudo ufw enable`  
  From your local terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@3.8.126.57`


## Configure the local timezone to UTC:
  - Run `sudo dpkg-reconfigure tzdata` and then choose UTC


## Create a new user 'grader':
 1. While logged in as ubuntu, add user: `sudo adduser grader`

## Give grader the permission to sudo:
 1. Run `sudo visudo` add grader  `ALL=(ALL:ALL) ALL` , Save and exit.
 2. Verify that grader has sudo permissions. Run `su - grader`, and run `sudo -l`.


## Create an SSH key pair for grader using the ssh-keygen:
1. generate keys on local machine using`ssh-keygen` , then save the private key in `~/.ssh` on local machine.
2. Two files will be generated ( ~/.ssh/keys and ~/.ssh/keys.pub)
3. Run `sudo cat ~/.ssh/keys.pub` and copy the contents of the file.
4. On the grader's VM run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit.
5. Run `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
6. Run `sudo nano /etc/ssh/sshd_config`  `PasswordAuthentication no` and `permitRootLogin no`
7. Run `sudo service ssh restart`
8. On the local machine, run: `ssh -i ~/.ssh/keys -p 2200 grader@3.8.126.57`.


## Install Apache:
  - `sudo apt-get install apache2`

## Install mod_wsgi:
  1. Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  2. Enable mod_wsgi with `sudo a2enmod wsgi`
  3. Start the web server with `sudo service apache2 start`

  
## Clone the Catalog app from Github:
  1. Install git using: `sudo apt-get install git`
  2. `cd /var/www`
  3. `sudo mkdir catalog`
  4. Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
  5. `cd /catalog`
  6. Clone your project from github `git clone https://github.com/Saliha21/Item-catalog.git catalog`
  7. `cd /var/www/catalog` Create a catalog.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/catalog/")
  sys.path.insert(1, "/var/www/catalog/")

  
  from catalog import app as application
  application.secret_key = 'super_secret_key'
  ```
  - Rename `project.py to __init__.py`   `mv project.py __init__.py`
  
## Install virtual environment:
  1. Install the virtual environment `sudo pip install virtualenv`
  2. Create a new virtual environment with `sudo virtualenv venv`
  3. Activate the virutal environment `. venv/bin/activate`
  4. Change permissions `sudo chmod -R 777 venv`
  5. Deactivate the virtual environment  `deactivate`.

### Install the following dependencies:
- Install pip with `sudo apt-get install python-pip`
   ```
       pip install httplib2
       pip install requests
       pip install --upgrade oauth2client
       pip install sqlalchemy
       pip install flask
       sudo apt-get install libpq-dev
       pip install psycopg2
       ```

## Configure and enable a new virtual host:
Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
- Paste this code: 

  ```
  <VirtualHost *:80>
      ServerName 3.8.126.57
      ServerAlias ec2-3-8-126-57.eu-west-2.compute.amazonaws.com
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
  


- Enable the virtual host `sudo a2ensite catalog`

## Install and configure PostgreSQL:
1. `sudo apt-get install libpq-dev python-dev`

2. `sudo apt-get install postgresql postgresql-contrib`

3. `sudo su - postgres` and run `psql`

4. `CREATE USER catalog WITH PASSWORD 'catalog';`

5. `ALTER USER catalog CREATEDB;`

6. `CREATE DATABASE catalog WITH OWNER catalog;`

- `\c catalog`

- `REVOKE ALL ON SCHEMA public FROM public;`

- `GRANT ALL ON SCHEMA public TO catalog;`

- `\q` and `exit`


7. Edit create engine line in your `__init__.py` , `database_setup.py` and `db.py` to: 
`engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`

8. Run `python database_setup.py`


## Update path of client_secrets.json file:
1. `nano __init__.py`

2. Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
  
  
# Helpful Resources:


**Thanks to *(https://github.com/bencam/linux-server-configuration)* and *(https://github.com/anumsh/Linux-Server-Configuration)* for a very helpful README**
