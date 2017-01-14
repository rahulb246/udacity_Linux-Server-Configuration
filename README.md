#Linux Server Configuration

###Server Info
  Public IP : 35.167.180.128
  URL : http://35.167.180.128
  Port : 2200

###Software Installed

  Apache2
  PostgreSQL
  bleach
  dict2xml
  git
  github-flask
  httplib2
  libapache2-mod-wsgi
  oauth2client
  python-flask
  python-pip
  python-psycopg2
  python-sqlalchemy
  requests

###Configuration

  Update all currently installed packages

  sudo apt-get update
  sudo apt-get upgrade

###Create a new user named grader

  adduser grader

###Give the user grader permission to sudo

  echo "grader ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/grader

###Set up SSH Authentication

  Generate SSH key pairs in local machine and copy the .pub file using ssh-copy-id command

###Change the SSH port from 22 to 2200

  sudo nano /etc/ssh/sshd_config
  Change Port 22 to Port 2200

###Remote login of the root user has been disabled

  sudo nano /etc/ssh/sshd_config
  Ensure PermitRootLogin has a value no
  Ensure PasswordAuthentication has a value no

###Restart SSH service

  sudo service ssh restart

###Configure the Uncomplicated Firewall (UFW)

  Block all incoming requests

  sudo ufw default deny incoming
  Allow all outgoing requests

  sudo ufw default allow outgoing
  Allowing incoming connections for SSH (port 2200)

  sudo ufw allow 2200/tcp
  Allowing incoming connections for HTTP (port 80)

  sudo ufw allow www
  Allowing incoming connections for NTP (port 123)

  sudo ufw allow ntp
  Enable ufw

  sudo ufw enable

###Configure the local timezone to UTC

  Reconfiguring the tzdata package

  sudo dpkg-reconfigure tzdata

  select `None of the above` then `UTC`

###Install and configure Apache to serve a Python mod_wsgi application

  sudo apt-get install apache2
  sudo apt-get install libapache2-mod-wsgi

###Install and configure PostgreSQL

  sudo apt-get install postgresql

###Create a new user named catalog that has limited permissions to your catalog application database

  sudo -i -u postgres

###Create new dastbase user catalog

  postgres@server:~$ createuser --interactive -P
  Enter name of role to add: catalog
  Enter password for new role:
  Enter it again:
  Shall the new role be a superuser? (y/n) n
  Shall the new role be allowed to create databases? (y/n) n
  Shall the new role be allowed to create more new roles? (y/n) n
  Create catalog Database

  postgres:~$ psql
  CREATE DATABASE catalog;
  \q
  exit

###Install git, clone and setup Catalog App project

  sudo apt-get install git
  clone repo

###Protect .git directory
  create a .htaccess file in .git folder and add the following

  <Directory .git>
      order allow,deny
      deny from all
  </Directory>

###Install application dependences

  sudo apt-get install python-psycopg2
  sudo apt-get install python-flask
  sudo apt-get install python-sqlalchemy
  sudo apt-get install python-pip
  sudo pip install bleach
  sudo pip install dict2xml
  sudo pip install github-flask
  sudo pip install httplib2
  sudo pip install oauth2client
  sudo pip install requests

###Configure and Enable New Virtual host

  Create a new configuration file:

  sudo nano /etc/apache2/sites-available/catalog.conf
  Add this code to catalog.conf:

  <VirtualHost *:80>
              ServerName 35.167.180.128
              WSGIScriptAlias / /var/www/catalog/application.wsgi
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
  Enable the virtual host:

  sudo a2ensite catalog
  Create a .wsgi file:

  cd /var/www/catalog
  sudo nano catalog.wsgi
  Add code to file:

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")
  from catalog import app as application
  application.secret_key = 'Add your secret key'

###Update Database connection string where ever used to the following:

  postgresql://catalog:catalog@localhost/catalog

###create database schema and populate the database

  python database_setup.py
  python populate_db.py

###Modify the oauth tokens

###Restart Apache
  sudo service apache2 restart

###visit the URL to find the running application
