# Linux-server-Configuration
### About the project

> A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers

- IP Address: 3.10.209.10
- SSH Port:2200

### Steps Followed to Configure the server

#### 1. Update all packages

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

Enable automatic security updates

```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

#### 2. Change time zone to UTC and Fix language issues

```
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8
```

#### 3. Create a new user grader and Give him `sudo` access

```
sudo adduser grader
sudo nano /etc/sudoers.d/grader 
```

Then add the following text `grader ALL=(ALL) ALL`

#### 4. Setup SSH keys for grader

- On local machine `ssh-keygen` Then choose the path for storing public and private keys
- On remote machine home as user grader

```
sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys 
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys 
nano .ssh/authorized_keys 
```

Then paste the contents of the public key created on the local machine

#### 5. Change the SSH port from 22 to 2200 | Enforce key-based authentication | Disable login for root user

```
sudo nano /etc/ssh/sshd_config
```

Then change the following:

- Find the Port line and edit it to 2200.
- Find the PasswordAuthentication line and edit it to no.
- Find the PermitRootLogin line and edit it to no.
- Save the file and run `sudo service ssh restart`

#### 6. Configure the Uncomplicated Firewall (UFW)

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw enable
```

#### 7. Install Apache2 and mod-wsgi for python3 and Git

```
sudo apt-get install apache2 libapache2-mod-wsgi git
```

#### 8. Install and configure PostgreSQL

```
sudo apt-get install libpq-dev python3-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
```

Then

```
CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog WITH OWNER continent;
\c continent
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO continent;
\q
exit
```

**Note:** change the engine in the project.py 

```
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```

#### 9. Clone the Catalog app from GitHub and Configure it

```
cd /var/www/
sudo mkdir catalog
sudo chown grader:grader catalog
git clone <your_repo_url> catalog
cd catalog
git checkout production # If you have a diffrent branch!
nano catalog.wsgi
```

Then add the following in `catalog.wsgi` file

```
#!/usr/bin/python
import sys
sys.stdout = sys.stderr

# Add this if you'll create a virtual environment, So you need to activate it
# -------
activate_this = '/var/www/catalog/env/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))
# -------

sys.path.insert(0,"/var/www/project_2")

from app import app as application
```

#### 10. install requirment.txt

```
sudo apt-get install python-pip
sudo -H pip install virtualenv
virtualenv env
source env/bin/activate
pip3 install -r requirement.txt
```

#### 11. Configure apache server

```
sudo nano /etc/apache2/sites-available/project.conf
```

Then add the following content:

```
# serve catalog app
<VirtualHost *:80>
  ServerName <IP_Address or Domain>
  ServerAlias <DNS>
  ServerAdmin <Email>
  DocumentRoot /var/www/project_2
  WSGIDaemonProcess project_2 user=grader group=grader
  WSGIScriptAlias / /var/www/project_2/project_2.wsgi

  <Directory /var/www/project_2>
    WSGIProcessGroup project_2
    WSGIApplicationGroup %{GLOBAL}
    Require all granted
  </Directory>
```

#### 14. Reload & Restart Apache Server

```
sudo service apache2 reload
sudo service apache2 restart
```

