# Linux setup project
A baseline installation of a AWS lightsail out of the box Linux distribution was prepared to host a web applications, after installing updates, securing it from a number of attack vectors and installing/configuring Apache Web and PostGres database servers to run the web application data.

# Details on the Linux Box

### IP address and SSH port
18.188.61.208 port 2200

### Complete URL to the hosted web application
http://18.188.61.208.xip.io

### Summary of software installed and configuration changes made.
* Provisioned LighSail instance from https://lightsail.aws.amazon.com/ls/webapp/home/instances
* Updated the software packages using apt-get package manager repository.
  ``` linux
    sudo apt-get update
    sudo apt-get upgrade
    ```
* Hardened the server against common attack vectors
* Disallowed remote root login, changed default ssh port, and enforced RSA key authorizations.
  ```linux
    sudo nano sshd_config
      Port 2200
      PermitRootLogin no
      PasswordAuthentication no
    sudo service sshd restart
    ```
* Opened only the required ports - ssh, www and ntp
```linux

    sudo ufw status
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow 2200
    sudo ufw allow 5000
    sudo ufw allow 80/tcp
    sudo ufw allow 123/tcp
    sudo ufw enable
```
*  Created an account for udacity grader with sudo privileges.  Private key will be submitted as part of submission.
```linux
    sudo adduser grader
    sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
    sudo nano /etc/sudoers.d/grader
```
* Added RSA login key 
```linux
    ssh-keygen
    udacitygrader.pub
    
    cd ~grader
    sudo mkdir .ssh
    sudo chmod 700 .ssh
    sudo touch .ssh/authorized_keys
    sudo chmod 644 .ssh/authorized_keys
    sudo nano .ssh/authorized_keys
```
* Setup Web Server
  ```linux
    sudo apt-get install apache2
    ```
    http://18.188.61.208
    It works!

    sudo -H apt-get install libapache2-mod-wsgi

* Installed Python and required softare using pip install
  ```linux
    sudo -H apt update	            --refresh repo
    sudo -H apt dist-upgrade       --update all software
    sudo -H apt install python2.7 python-pip
    sudo -H pip install Flask
    pip install --upgrade pip
    sudo -H apt-get install sqlite3 libsqlite3-dev
    sudo -H pip install sqlalchemy
    sudo -H pip install --upgrade oauth2client
    sudo -H pip install requests
```
* Installed PostGres database server
  ```linux
    sudo apt-get install postgresql postgresql-contrib
    sudo -u postgres psql postgres
    \password postgres
```
* Verify only local connections are allowed
    sudo nano /etc/postgresql/9.5/main/pg_hba.conf 

* Created Application DB user account and database
```sql
    CREATE USER catalog WITH
      LOGIN
      NOSUPERUSER
      CREATEDB
      NOCREATEROLE
      INHERIT
      NOREPLICATION
      CONNECTION LIMIT -1
      PASSWORD 'catalog';
    REVOKE ALL ON SCHEMA public FROM public;
    GRANT ALL ON SCHEMA public to public;
    CREATE DATABASE itemcatalog
        WITH 
        OWNER = catalog
        ENCODING = 'UTF8'
        CONNECTION LIMIT = -1;
```
    su - catalog
    psql itemcatalog

* Use Python to add tables and data.
    python database_setup.py
    python lotsofmenus.py

* Install GIT
sudo -H apt-get install git


* Configure console.google.com to add the new paths for oauth
    http://18.188.61.208.xip.io
    http://18.188.61.208.xip.io/login   redirect uri
    http://18.188.61.208.xip.io/gconnect redirect uri


* Configure Mod WSGI - Web Application Gateway to Python
  cd /var/www
  sudo mkdir /var/www/itemcatalogapp
  sudo mkdir /var/www/itemcatalog.com
  sudo mkdir /var/www/itemcatalog.com/logs

  sudo nano /var/www/itemcatalogapp/itemcatalog.wsgi
  Add the following:

  import sys
  sys.path.insert(0,"/var/www/itemcatalogapp/")
  from itemcatalog import app as application


* Test if Flask is working
  sudo touch /var/www/itemcatalogapp/itemcatalog.py
  sudo nano /var/www/itemcatalogapp/itemcatalog.py
  add following:
  ```python
  from flask import Flask
  app = Flask(__name__)
  @app.route("/")
  def hello():
      return "Hello, Flask!"
  if __name__ == "__main__":
      app.run()
```
* Configure Mod WSGIU
    cd /etc/apache2/sites-enabled
    sudo nano mv /etc/apache2/sites-enabled/000-default.conf  /etc/apache2/sites-enabled/001-
    default.conf
    sudo nano /etc/apache2/sites-enabled/000-default.conf   --itemcatalog.com.conf

configure mod_wsgi to start running this python application
on root url /
set default folder location to look for python files
```html
<VirtualHost *:80>
      ServerAdmin sherin.kuruvilla@gmail.com
      ServerName itemcatalog.com
      ServerAlias itemcatalog.com
      ErrorLog /var/www/itemcatalog.com/logs/error.log
      CustomLog /var/www/itemcatalog.com/logs/access.log combined

      WSGIDaemonProcess itemcatalogapp user=www-data group=www-data threads=5
      WSGIProcessGroup itemcatalogapp
      WSGIScriptAlias / /var/www/itemcatalogapp/itemcatalog.wsgi
      Alias /static/ /var/www/itemcatalog/static
      <Directory /var/www/itemcatalogapp>
    WSGIProcessGroup itemcatalogapp
    WSGIApplicationGroup %{GLOBAL}
          Order allow,deny
          Allow from all
      </Directory>
    </VirtualHost>
```
change owner:
    sudo chown -R www-data:www-data /var/www/itemcatalog.com

reload apache configuration
    sudo /etc/init.d/apache2 reload

    sudo a2ensite itemcatalog.com.conf
    sudo a2dissite 000-default.conf
    systemctl status apache2.service

* veriy mod wsgi is working
http://18.188.61.208.xip.io
 It works!   (Hello, Flask!)

* Clone project from git to Ubuntu
```linux
  cd /var/www
  sudo mv itemcatalogapp test
  sudo git clone "https://github.com/sherinkuruvilla/OAuth2.0" itemcatalogapp
  sudo cp /var/www/test/itemcatalog.wsgi /var/www/itemcatalogapp/itemcatalog.wsgi   --added to git
  sudo /etc/init.d/apache2 reload
```

* Application URL is working!
```html
  http://18.188.61.208.xip.io
```


### A list of any third-party resources you made use of to complete this project
1. http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
2. http://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html
3. https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-apache-and-mod_wsgi-on-ubuntu-16-04
4. Several forums and google results
