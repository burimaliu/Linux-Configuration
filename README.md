# Linux Configuration

## Configure and setup python application

### About server
- IP `52.25.17.138`
- Port `2200`
- Online Application http://52.25.17.138/

### Add user / add to sudo group

Add a user grader
`adduser grader`

Add the user to the sudo group
`adduser grader sudo`

Create a directory `.ssh` inside `/home/grader/`
- `mkdir /home/grader/.ssh`
- `chown grader /home/grader/.ssh`
- `chmod 700 /home/grader/.ssh`

Copy the `/root/.ssh/authorized_keys` file to `/home/grader/.ssh/` directory
- `cp /root/.ssh/authorized_keys /home/grader/.ssh/`
- `chmod 600 authorized_keys`
- `chown grader authorized_keys`

### DISABLE root remote login
- Set `permitRootLogin` in `/etc/ssh/sshd_config` to `no`
- Add a new line with `AllowUsers grader`
- restart ssh

### UPDATE all packages
`apt-get update`

### Setup Automatic Updates
- `sudo apt-get install unattended-upgrades` Install's `unattended-upgrades` package
- `sudo dpkg-reconfigure unattended-upgrades` Select `Yes`
```
This package can download and install security upgrades automatically
and unattended, taking care to only install packages from the
configured APT source
```

### RUN ssh daemon on port 2200:
- Set 'Port' in /etc/ssh/sshd_config to '2200'
Port 2200
- Restart the ssh daemon
- restart ssh

### SET local time to UTC
`dpkg-reconfigure tzdata`
- select 'Others' or 'None of the above'
- select 'UTC' 

### ENABLE universal firewall for given ports
- `sudo ufw allow 2200`
- `sudo ufw allow 80`
- `sudo ufw allow 123`
- `sudo ufw enable`
- check the active rules:
- `sudo ufw status verbose`

### (Optional) Install Fail2Ban to protect from bruteforce attacks
- `sudo apt-get install fail2ban`
- `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
-  (Optional) update configuration here: `sudo nano /etc/fail2ban/jail.local`
-  Start it `sudo service fail2ban start` 


### INSTALL webserver and WSGI
- install Apache 2
- `sudo apt-get install apache2`
- #### you can check now your Apache server working at http://THE.VM.IP.NUMBER
- install mod_wsgi
- `sudo apt-get install python-setuptools libapache2-mod-wsgi python-dev`
- `sudo service apache2 restart`
- #### Check if wsgi is enabled
- `sudo a2enmod wsgi `

### INSTALL POSTgre
- #### install the package
- `sudo apt-get install postgresql`
- #### check if service is running
- `sudo service postgresql status`

### POSTgre management
- login as postgres user into psql to have administrative rights on the database
- `su - postgres`
- `psql`
- `CREATE DATABASE catalog;`
- `CREATE USER catalog PASSWORD 'somepwd';`
- `GRANT ALL ON DATABASE catalog TO catalog;`

### INSTALL git
- `sudo apt-get install git`

### CLONE the app from GIT
- `cd /var/www/FlaskAapp/FlaskApp/ItemCatalog`
- `sudo git clone https://github.com/burimaliu/Item-Catalog.git`

### Install PSYCOPG2
- #### substitute X.X with your POSTgre version
- `sudo apt-get install postgresql-server-dev-X.X`
- `sudo pip install psycopg2==2.6`

### RUN flask
- `cd /var/www/FlaskAapp/FlaskApp/ItemCatalog`
- #### install dependencies
- `sudo sh pg_config.sh`
- #### edit configuration file and update database configuration 
- ` sudo nano init.sh`
- #### Apply new configuration
- `sudo sh init.sh`
- rename `application.py` to `__init__.py`

### create apache vhost
- `nano /etc/apache2/sites-available/FlaskApp.conf`

```
<VirtualHost *:80>
ServerName IP
ServerAdmin admin@IP
WSGIScriptAlias / /var/www/FlaskApp/FlaskApp/ItemCatalog/flaskapp.wsgi
<Directory /var/www/FlaskApp/FlaskApp/ItemCatalog/>
#WSGIApplicationGroup %{GLOBAL}
Order deny,allow
Allow from all
<Files flaskapp.wsgi>
Require all granted
</Files>
</Directory>
ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn
CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Create wsgi file
- `nano /var/www/FlaskApp/FlaskApp/ItemCatalog/flaskapp.wsgi`

```
#!/usr/bin/python
import os
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/FlaskApp/ItemCatalog/")
from __init__ import app as application
#application.secret_key = 'Add your secret key'
```
### Restart web server
- `sudo service apache2 restart`

### (Optional) Application availability status
- Create a vhost `sudo nano /etc/apache2/mods-available/status.conf`
```
<Location /server-status>
	SetHandler server-status
	Order deny,allow
	Deny from all
	Allow from 127.0.0.1 ::1
	Allow from YOUR-IP
</Location>
```
- `sudo apache2 restart`
- Browse YOUR-IP/server-status
