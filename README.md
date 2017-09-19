# Linux Server Configuration

Hosting a web application on a Linux Server starting from a base installation.

> #### TL;DR
> Application can be found at http://ec2-35-164-101-52.us-west-2.compute.amazonaws.com
> 
> - IP: 35:164:101:52
> - SSH PORT: 2200
>
> _This link will expire on or before October 28_

## Walkthrough

> #### Key issues I had:
>
> - Attempting to use sqlite...hahaha
> - Forgetting to change the SECRET_KEY from a dynamic variable to a fixed GUID

### 1: Updates/Upgrades for the packages
    
1. Update packages and their versions:  
  `sudo apt-get update`
2. Install updates:  
  `sudo sudo apt-get upgrade`

### 2: Grader User

1. Create grader user: `adduser grader`
2. Give new user the permission to sudo
  - Open the sudo configuration: `visudo`
  - Add the user below root `root ALL...`: `NEWUSER ALL=(ALL:ALL) ALL`
    - I know this is dangerous due to updates wiping this, but for this project, I did it :(
3. Deal with `unable to resolve host` by doing the following:
  - `sudo nano /etc/hosts`
  - add `127.0.1.1 ip-10-20-52-12`
  
### 3: Change the default SSH port
1. Open the config file: `sudo nano /etc/ssh/sshd_config` 
2. Change to Port 2200.
3. Change `PermitRootLogin` from `without-password` to `no`.
4. Append `UseDNS no`.
5. Append `AllowUsers grader`.
6. Restart SSH Service:  `/etc/init.d/ssh restart` 

### 4: Setup firewall
- `sudo ufw allow 2200/tcp`
- `sudo ufw allow 80/tcp`
- `sudo ufw allow 123/udp`
- `sudo ufw enable`

### 5: Configure the local timezone to UTC
1. Open the timezone selection dialog:
  `$ sudo dpkg-reconfigure tzdata`
2. Then chose 'None of the above', then UTC.
3. *Setup the ntp daemon ntpd for regular and improving time sync:  
  `$ sudo apt-get install ntp`

### 6: Install Apache/WSGI and PostgreSQL

1. `sudo apt-get install apache2`
2. `sudo apt-get install libapache2-mod-wsgi`
3. `sudo apt-get install postgresql postgresql-contrib`

#### Create a PostgreSQL user called `catalog` with:

1. `sudo -u postgres createuser -P catalog`
2. `sudo -u postgres createdb -O catalog catalog`
    - Will populate this with `model.py` later

### 7: Install Flask, SQLAlchemy, etc
Issue the following commands:
```
sudo apt-get install git
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install -r requirements.txt
```

### 8: Clone the Asset-Catalog repository
```
cd /var/www/catalog/
sudo mkdir Asset-Catalog
git clone https://github.com/SteveWooding/fullstack-nanodegree-vm.git Asset-Catalog
```

### 9: Update the catalog.wsgi
```python

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application

```


### 10: Configure Apache2 to serve the app
Update the `catalog.conf` file with the proper information to server the app
```xml
<VirtualHost *:80>
      ServerName 35.164.101.52
      ServerAlias ec2-35-164-101-52.us-west-2.compute.amazonaws.com
      ServerAdmin jfukura@gmail.com
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

To make these Apache2 configuration changes live, reload Apache:

`sudo service apache2 reload`

Application is now available at: http://ec2-35-164-101-52.us-west-2.compute.amazonaws.com/
