## Udacity | Full Stack Web Developer | Project 3 : Linux Server Configuration

##### Created by: Ebtihal Abduallah


### Introduction:
 Project focus in installation of a Linux server and prepare it to host web applications and will secure server from a number of attack vectors, learn how to configure firewall, dealing with encrypted key as login secure method, install and configure a database server, and deploy one of existing web applications onto it.


### Server information:
**Public IP address:** 45.33.56.207
**DNS name:** www.techitemlist.com
**Operating System:** Ubuntu 16.04 LTS
**Key:** ssh -i ~/.ssh/authorized_keys -p 2200 grader@45.33.56.207
**SSH port:** 2200


### Installation:
Server Hosted in www.linode.com for more information [click here](https://docs.google.com/presentation/d/1SQxIiiso372u04SenxKxoXgxY07doUbGawxJWOUhiFU/edit?usp=sharing)


### Configuration:
#### Step 1:
* Update installed packages  
```
sudo apt-get update  
sudo apt-get upgrade
```
* Configure Uncomplicated Firewall (ufw), Allow port SSH:2200, HTTP:80, NTP:123, HTTPS:443
```
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw allow 443/tcp
sudo ufw enable
```
**Note:**
* If you receive error "ufw : command not found" install ufw 
`sudo apt install ufw`
*  To be sure firewall activate 
 `sudo ufw status` 
* Change the port from 22 to 2200  
 `sudo nano /etc/ssh/sshd_config`
 **Note:** after change port to 2200 run:  
 `sudo service ssh restart`
* Configure Local Timezone to UTC
 `sudo dpkg-reconfigure tzdata` 
 first choose -non of the above- then brows the list choose **UTC** then ok 

#### Step 2: 
* Create new user grader
 `sudo adduser grader`
* Give grader permission to sudo Create a new file in the sudoers directory  
 `sudo nano /etc/sudoers.d/grader`
* add the line and save the file 
 `grader ALL=(ALL:ALL) NOPASSWD:ALL`
* Create an SSH key for grader
On terminal run the command to generate key pair
 `sudo ssh-keygen -t rsa`
  - It will ask about file name the default (id_rsa) press enter 
  - Then it will ask to add passphrase (you can leave it empty or you can write word to make it more secure if any one have your private key should use passphrase to use it)
  - copy the public key content id_rsa.pub 
    `cat ~/.ssh/id_rsa.pub`
  - Note: copy the string and exclude  the email in the last row
  - create .ssh directory in grader account 
    `mkdir /home/grader/.ssh`
  - Past coped key info in **authorized_keys** file and save it 
  `sudo nano  /home/grader/.ssh/authorized_keys` 
  - Change permission to ssh folder 
   `chmod 700 /home/grader/.ssh`
    `chmod 644 /home/grader/.ssh/authorized_keys`
  - Change .ssh folder owner
    `sudo chown -R grader:grader /home/grader/.ssh` 
  - Restart SSH service 
    `sudo service ssh restart`

  **Logout then login thow ssh key : ssh -i ~/.ssh/authorized_keys -p 2200 grader@45.33.56.207**
 
 **Note:**
 To enforce key authentication from the ssh configuration file by editing 
  `sudo nano /etc/ssh/sshd_config` 
 look for line that says **PasswordAuthenticatio**n and change it to **no**
 
#### Step 3:
* Install Apache 
 `sudo apt-get install apache2`
* Configure Apache mod_wsgi 
  `sudo apt-get install libapache2-mod-wsgi python-dev`
* Enable mod_wsgi 
 `sudo a2enmod wsgi`
* Start web server  
 `sudo service apache2 start`

#### Step 4:
* Install git 
 `sudo apt-get install git`
* Configure global variable
```
  git config --global user.email "you@example.com"
  git config --global user.name "Your UserName"
```
* Create project folder 
 `sudo mkdir /var/www/catalog`
* Change owner of the newly created catalog folder
 `sudo chown -R grader:grader catalog`

* Clone your project file inside it 
 `cd /var/www/catalog`
 `git clone <repository link> catalog` 
* Create file **catalog.wsgi** with code below 
  `sudo nano /var/www/catalog/catalog.wsgi`
 code:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import app as application
application.secret_key = 'super_secret_key'
```

**Note:**
To make .git directory not accessible 
  `cd /var/www/catalog/catalog/.git`
Create a **.htaccess** file in the **.git** folder and put the following in this file:
```
Order allow,deny
Deny from all
```
#### Important: 
* in __init__.py python file update Google Sing-in JSON file path in 2 line
 ```
1: CLIENT_ID = json.loads(open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
2:  Gconnect function
oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope='')
```
* Check python package version if 2.7 be sure the sheebang  line   `#!/usr/bin/env python2.7`
* Update DB engine
```
create_engine('postgresql://catalog:password@localhost/catalog')
```
**Note:** Also updated in database_setup.py and seeder file data.py 
* Update main method   
```
app.run(host='Public_Ip address', port=80)
```
#### Step 4:
* Install virtual environment
  - First install pip with this command
  `sudo apt-get install python-pip`
  - Install the virtual environment
  `sudo pip install virtualenv`
* Create a new virtual environment 
 `cd /var/www/catalog`
 `sudo virtualenv venv`
* Activate it 
 `source venv/bin/activate`
* Change permissions venv folder 
 `sudo chmod -R 777 venv`
* Configure a new virtual host
  - Create a new file with this : 
    `sudo nano /etc/apache2/sites-available/catalog.conf`
write the code below
```
<VirtualHost *:80>
    ServerName 35.234.117.82
    ServerAlias
    ServerAdmin admin@35.234.117.82
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog	
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

* Enable the virtual host and disable 000-default.conf then reload apache2 :
```
sudo a2ensite catalog
sudo a2dissite 000-default.conf 
service apache2 reload 
```

#### Step 5:
* Install Flask and other important packages
```
sudo Pip install Flask
sudo pip install SQLAlchemy ==1.2.13
sudo pip install Flask-Login , Flask-KVSession==0.6.2 , Jinja2, Werkzeug, httplib2, oauth2client, requests
Simplekv, wsgiref, python-gflags, itsdangerous, google-api-python-client
Sudo Pip install psycopg2-binary
```

#### Step 6:
* Install PostgreSQ package:
 `sudo apt-get install postgresql postgresql-contrib`
* Configure PostgreSQL:
 `sudo -u postgres psql postgres`
* Create database user catalog – limited permission-
```
postgres=# CREATE USER catalog WITH PASSWORD ‘passwor’;
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
```

* Run database file 
  `python /var/www/catalog/catalog/database_setup.py`
* Run seeder file
  `python /var/www/catalog/catalog/data.py`

* Restart server
 `sudo service apache2 restart`
* Run python file
 `sudo python __init__.py`


##### Create DNS name refers to your instance IP Address
After register DNS Name do the following:
* Update **catalog.conf** file
```
    ServerName www.techitemlist.com
    ServerAlias techitemlist.com
    ServerAdmin admin@techitemlist.com
```
* Update Google Sign-in Credential 
 **Authorised domains** - **Authorised JavaScript origins** -
 **Authorised redirect URIs** 
**<web link>/login** 
**<web link>/gconnect** 
* Download updated JSON file add it to project folder **/var/www/catalog/catalog**

**Note:**
 Update **sshd_config** to prevent any unsecure usage for root user 
* Change **PermitRootLogin** To **no** 
  `sudo nano /etc/ssh/sshd_config`
* Restart SSH 
   `sudo service ssh restart`



#### Resource: 
* [APACHE HTTP](https://httpd.apache.org/docs/2.4/vhosts/name-based.html)
* [Deploy a Flask application](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [mod_wsgi APACHE](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)
* [Troubleshooting APACHE issues](https://www.linode.com/docs/troubleshooting/troubleshooting-common-apache-issues/)
* [Python virtual environments](https://modwsgi.readthedocs.io/en/develop/user-guides/virtual-environments.html)
* [PostgreSQL Ubuntu documentation](https://help.ubuntu.com/community/PostgreSQL)




















