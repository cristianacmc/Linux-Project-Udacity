# Linux Server Configuration

This is the last project of Udacity nanodegree Full Stack Web Developer where I have to deploy the web aplication Catalog in a Linux server instance.
The focus of the project is to teach the students how to secure and set up a Linux server.

## List of resources used:

- Google API
- Apache2
- PostgreSQL
- Amazon Lightsail
- git
- Python
- SQLAlchemy
- Flask

## Access to the Server

- IP address: 34.242.35.57
- SSH port: 2200
- Web application URL: http://34.242.35.57 or http://34.242.35.57.xip.io
* In order to log you in, please use http://34.242.35.57.xip.io

### Get the Server:

I chose Amazon Lightsail to be the linux server and set the following configurations to create the instance:

- Set `Ubuntu` as instance image and **Os Only**
- Chose the instance plan;
- Set the name of the instance as **Linux-Project**

#### Log into the instance with SSH:

Once your instance has started up, the public IP address is displayed along with its name.
In order to run the ssh command to access remotely the server, I followed these steps:

- Download the instance private key
- Move the file to ~/.ssh/ and rename it to lightsailkey.rsa
- Run chmod `600 ~/.ssh/lightsailkey.rsa`
- Log into the instance with: `~/.ssh/lightsailkey.rsa ubuntu@34.242.35.57`

### Secure your server

- Update all currently installed packages:
1. Check the updates available: `sudo apt-get update`
2. Download and update the packages: `sudo apt-get upgrade`

### Configure the Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):

1. Block all the incoming requests: `sudo ufw default deny incoming`
2. For the outgoing connections: `sudo ufw default allow outgoing`
3. For the server to support SSH on port 2200:
- `sudo ufw allow ssh`
- `sudo ufw allow 2200/tcp`
4. To support HTTP server: `sudo ufw allow www`
5. To allow NTP: `sudo ufw allow 123/udp`
6. Deny the default port 22: `sudo ufw deny 22`
7. run `sudo service ssh restart`
8. Enable the firewall: `sudo ufw enable`
9. On **Amazon Lightsail/Linux_project/network**, configure the Firewall rules to match the configurations above

### Give grader access

1. Log into the server using ssh: `ssh -i ~/.ssh/lightsailkey.rsa -p2200 ubuntu@34.242.35.57`
2. Create a new user account named grader:
- `sudo adduser grader`
- Set the password `grader`
3. Give grader the permission to sudo:
- `sudo touch /etc/sudoers.d/grader`
- `sudo nano /etc/sudoers.d/grader`
- Add `grader ALL=(ALL) NOPASSWD:ALL`
4. Create an SSH key pair for grader using the ssh-keygen tool.
- run `ssh-keygen` on the local machine
- I gave the name to the file _graderKey_
- Entered the passphrase
- Logged in the server as grader
- Created a directory `.ssh`
- Created a file to store all of the public keys within this directory called `authorized_Keys`
- On the local machine, copied the content of graderKey.pub
- Back to the terminal logged as grader, paste the content to .ssh/authorized_keys file
- run chmod 700 on ssh directory
- run chmod 644 on the authorized_keys file

To log as grader use the following command: `ssh -i ~/.ssh/graderKey -p 2200 grader@34.242.35.57`

### Configure the local timezone to UTC

- `sudo dpkg-reconfigure tzdata`
- `None of the above/UTC`

### Install and configure Apache to serve a Python mod_wsgi application

- Installed apache - `sudo apt-get install apache2`
- Install mod_wsgi - `sudo apt-get install libapache2-mod-wsgi python-dev`

### Install and configure PostgreSQL:

- `sudo apt-get install postgresql`

- Create a new database user named catalog that has limited permissions to your catalog application database:
1. ` sudo su - postgres`
2. `psql`
3. `CREATE ROLE catalog WITH LOGIN;`
4. `ALTER ROLE catalog CREATEDB;`
5. `\password catalog`

On the virtual machine create the user catalog:
1. `sudo adduser catalog`
2. `sudo touch /etc/sudoers.d/catalog`
3. `sudo nano /etc/sudoers.d/catalog`
4. Add `catalog ALL=(ALL) NOPASSWD:ALL`

### Clone and setup the Item Catalog project from the Github repository

On the virtual machine:
1. `sudo apt-get install git`
2. Create a directory catalog in `\var\www\`
3. Inside the directory `catalog` clone the project:
- `sudo git clone https://github.com/cristianacmc/itemCatalog.git`
4. `sudo chown -R ubuntu:ubuntu catalog/`
5. Inside the _itemCatalog_ directory:
- `mv application.py __init__.py`
6. where is `app.run(host='0.0.0.0', port=8000)` change to `app.run()`

Change the SQLite database to PostgreSQL in __init__.py, categories.py, database.py:
- `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`

### Authenticate login on Google API console

- On the Catalog Application, credentials, IDs do cliente OAuth 2.0, edit and add:
**Authorized JavaScript origins**
1. `http://34.242.35.57`
2. `http://34.242.35.57.xip.io`

**Authorized redirect URIs:**
1. `http://34.242.35.57.xip.io/login`
2. `http://34.242.35.57.xip.io/gconnect`
3. `http://34.242.35.57.xip.io/oauth2callback`

- Download the json file
- Copy and paste the json file content on itemcatalog/client_secrets.json
- On **itemCatalog/templates/login.html** add the client ID
- On __init__.py on 'client_secrets.json' change the path to:
`/var/www/catalog/itemCatalog/client_secrets.json`

**Install Dependencies on the virtual machine**
- `sudo apt-get install python-pipz`
- `sudo apt-get install python-virtualenv`
- on  **/var/www/catalog/itemCatalog/** run `virtualenv venv`
- `venv/bin/activate`
- `pip install sqlalchemy`
- `pip install --upgrade oauth2client`
- `pip install httplib2`
- `pip install requests`
- `pip install flask`
- `pip install psycopg2`
- `sudo apt-get install libpq-dev`

**Disable directories in the browser**

Create a file: /etc/apache2/sites-available/catalog.conf and add:
```
<VirtualHost *:80>
		ServerName 34.242.35.57
		ServerAdmin cristianacmc@gmail.com
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/itemCatalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/catalog/itemCatalog/static
		<Directory /var/www/catalog/itemCatalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Run:
- `a2ensite catalog`
- `sudo service apache2 reload`

**To serve the Flask application**

Create a file in /var/www/catalog/catalog.wsgi and add:
```
activate_this = '/var/www/catalog/itemCatalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")
from itemCatalog import app as application
application.secret_key = 'super_secret_key'
```
- Disable the Apache site: `sudo a2dissite 000-default.conf`

- In `/var/www` change the owner of the project:
`sudo chown -R www-data:www-data catalog/`

### Run the project:

- In **itemcatalog** directory, run virtualenv `. venv/bin/activate`
- `python categories.py`
- `deactivate`
- `sudo service apache2 restart`
- Open the browser with the url: http://34.242.35.57 or http://34.242.35.57.xip.io












