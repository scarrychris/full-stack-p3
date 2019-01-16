# linux-server-configuration
## Project Overview
To take a baseline installation of a Linux server and prepare it to host your web applications. To secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.
## Details specific to the server set up
* The IP address is [35.169.159.50](35.169.159.50)
* The SSH port used is 2200.
* The URL to the hosted webpage is:[http://35.169.159.50/](http://35.169.159.50/)
* grader password: redarg123
## Software Installed:
* Apache2
* mod_wsgi
* PostgreSQL
* git
* pip
* virtualenv
* httplib2
* Python Requests
* oauth2client
* SQLAlchemy
* Flask
* libpq-dev
* Psycopg2
* Feedparser
## Configuration steps
### Create an instance with Amazon Lightsail
* Sign in to [Amazon Lightsail](https://aws.amazon.com/lightsail/) using an Amazon Web Services account
* Follow the 'Create an instance' link
* Choose the 'OS Only' and 'Ubuntu 18.04 LTS' options
* Choose a payment plan
* Give the instance a unique name and click 'Create'
* Wait for the instance to start up
### Connect to the instance on your local machine
Note: While Amazon Lightsail provides a broswer-based connection method, this will no longer work once the SSH port is changed (see below). The following steps outline how to connect to the instance via the Terminal program on Mac OS machines (this can also be done on a Windows machine with a program such as PuTTY).
* Download the instance's private key by navigating to the Amazon Lightsail 'Account page'
* Click on 'Download default key'
* A file called LightsailDefaultPrivateKey.pem or LightsailDefaultPrivateKey-YOUR-AWS-REGION.pem will be downloaded; open this in your favorite text editor
* In terminal cd into your ```~/.ssh/``` directory and create a new file
```touch lightsail_key.rsa```
* Copy the text and put it in the file called lightsail_key.rsa in your local ```~/.ssh/ directory```
* Run ```chmod 600 ~/.ssh/lightsail_key.rsa```
* Log in with the following command: ```ssh -i ~/.ssh/lightsail_key.rsa ubuntu@35.169.159.50```, where 35.169.159.50 is the public IP address of the instance (note that Lightsail will not allow someone to log in as ```root```; ```ubuntu``` is the default user for Lightsail instances)
### Upgrade currently installed packages
* Check for what what package updates are available by running ```sudo apt update```
* Download & install available package updates by running ```sudo apt upgrade```
### Configure the firewall
* Start by changing the SSH port from ```22``` to ```2200``` 
* Run ```sudo nano /etc/ssh/sshd_config```(open up the /etc/ssh/sshd_config file, change the port number on line 5 to ```2200```, then restart SSH by running ```sudo service ssh restart```; restarting SSH is a very important step!)
* Check to see if the ufw (the preinstalled ubuntu firewall) is active by running ```sudo ufw status```
* Run ```sudo ufw default deny incoming``` to set the ufw firewall to block everything coming in
* Run ```sudo ufw default allow outgoing``` to set the ufw firewall to allow everything outgoing
* Run ```sudo ufw allow ssh``` to set the ufw firewall to allow SSH
* Run ```sudo ufw allow 2200/tcp``` to allow all tcp connections for port ```2200``` so that SSH will work
* Run ```sudo ufw allow www``` to set the ufw firewall to allow a basic HTTP server
* Run ```sudo ufw allow 123/udp``` to set the ufw firewall to allow NTP
* Run ```sudo ufw deny 22``` to deny port ```22``` (deny this port since it is not being used for anything. This is the default port for SSH, but this virtual machine has now been configured so that SSH uses port ```2200```)
* Also run ```sudo ufw deny 22/tcp``` to deny port ```22/tcp``` if it still persists.
* Run ```sudo ufw enable``` to enable the ufw firewall
* Run ```sudo ufw status``` to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:
```
To                         Action      From
--                         ------      ----
22/tcp                     DENY        Anywhere                  
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22                         DENY        Anywhere                  
22/tcp (v6)                DENY        Anywhere (v6)             
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)             
22 (v6)                    DENY        Anywhere (v6)
```
* Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports ```80```(TCP), ```123```(UDP), and ```2200```(TCP) should be allowed; make sure to delete the default SSH port ```22```)
* Now, to login (on a Mac), open up the Terminal and run:
```ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@35.169.159.50```, where 35.169.159.50 is the public IP address of the instance
Note: As mentioned above, connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port ```22```, which is now denied.
### Create a new user named ```grader```
* Run ```sudo adduser grader```
* Enter in a new UNIX password (twice) when prompted
* Fill out information for the new ```grader``` user
* To switch to the ```grader``` user, run ```su - grader```, and enter the password
### Give ```grader``` user sudo permissions
* Run ```sudo visudo```
* Search for a line that looks like this:
```root ALL=(ALL:ALL) ALL```
* Add the following line below this one:
```grader ALL=(ALL:ALL) ALL```
* Save and close the visudo file
* To verify that grader has sudo permissions, ```su``` as ```grader``` (run ```su - grader```), enter the password, and run ```sudo -l```; after entering in the password (again), the following should appear, meaning grader has sudo permissions:
```
Matching Defaults entries for grader on ip-172-26-8-80:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on ip-172-26-8-80:
    (ALL : ALL) ALL
```
### Allow ```grader``` to log in to the virtual machine
* Run ```ssh-keygen``` on your local machine
* Choose a file name for the key pair (such as ```grader_key```)
* Enter in a passphrase twice (two files will be generated; the second one will end in .pub)
* Log in to the virtual machine
* Switch to ```grader```'s home directory, and create a new directory called ```.ssh``` (run ```mkdir .ssh```)
* Run ```touch .ssh/authorized_keys```
* On your local machine, run ```cat ~/.ssh/insert-name-of-file.pub```
* Copy the contents of the file, and paste them in the ```.ssh/authorized_keys``` file on the virtual machine
* Run ```chmod 700 .ssh``` on the virtual machine
* Run ```chmod 644 .ssh/authorized_keys``` on the virtual machine
* Make sure key-based authentication is forced (log in as grader, open the ```/etc/ssh/sshd_config``` file, and find the line that says, ```# To disable tunneled clear text passwords, change to no here!```; if the next line says, ```PasswordAuthentication yes```, change the ```yes``` to ```no```; save and exit the file; run ```sudo service ssh restart``` to restart the SSH service and make this change active)
* Log in as the ```grader``` using the following command:
```ssh -i ~/.ssh/grader_key -p 2200 grader@35.169.159.50```
Note that a pop-up window will ask for ```grader``` user's passphrase.
### Configure the local timezone to UTC
* Run ```sudo dpkg-reconfigure tzdata```, and follow the instructions (UTC is under the 'None of the above' category)
* Test to make sure the timezone is configured correctly by running ```date```
### Install and configure Apache
* Run ```sudo apt install apache2``` to install Apache
* Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load
### Install mod_wsgi
* Install the ```mod_wsgi``` package (which is a tool that allows Apache to serve Flask applications) along with ```python-dev``` (a package with header files required when building Python extensions); use the following command:
```sudo apt install libapache2-mod-wsgi python-dev```
* Make sure mod_wsgi is enabled by running ```sudo a2enmod wsgi```
### Install PostgreSQL and make sure PostgreSQL is not allowing remote connections
* Install PostgreSQL by running ```sudo apt install postgresql```
* Open the PostgreSQL config file using ```sudo nano /etc/postgresql/10/main/pg_hba.conf```
* Make sure it looks like this:
```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```
### Make sure Python is installed
* Python should already be installed on a machine running Ubuntu 18.04. To verify, simply run ```python```. Something like the following should appear:
```
Python 2.7.15rc1 (default, Nov 12 2018, 14:31:15) 
[GCC 7.3.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```
### Create a new PostgreSQL user named catalog with limited permissions
* PostgreSQL creates a Linux user with the name postgres during installation; switch to this user by running ```sudo su - postgres``` (for security reasons, it is important to only use the postgres user for accessing the PostgreSQL software)
* Connect to psql (the terminal for interacting with PostgreSQL) by running ```psql```
* Create the catalog user by running ```CREATE ROLE catalog WITH LOGIN;```
* Next, give the catalog user the ability to create databases: ```ALTER ROLE catalog CREATEDB;```
* Finally, give the catalog user a password by running ```\password catalog```
* Check to make sure the catalog user was created by running ```\du```; a table of sorts will be returned, and it should look like this:
```
				   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 ```
* Exit psql by running ```\q```
* Switch back to the grader user by running ```exit```
### Create a Linux user called ```catalog``` and a new PostgreSQL database
1. Create a new Linux user called ```catalog```:
* run ```sudo adduser catalog```
* enter in a new UNIX password (twice) when prompted
* fill out information for ```catalog```
2. Give the ```catalog``` user sudo permissions:
* run ```sudo visudo```
* search for a line that looks like this: ```root ALL=(ALL:ALL) ALL```
* add the following line below this one: ```catalog ALL=(ALL:ALL) ALL```
* save and close the visudo file
* to verify that ```catalog``` has sudo permissions, su as catalog (run ```sudo su - catalog```), and run ```sudo -l```
* after entering in the UNIX password, the following should appear (meaning catalog has sudo permissions):
 ``` 
 Matching Defaults entries for catalog on ip-172-26-8-80:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User catalog may run the following commands on ip-172-26-8-80:
    (ALL : ALL) ALL
 ```
1. While logged in as catalog, create a database called catalog by running ```createdb catalog```
2. Run ```psql``` and then run ```\l``` to see that the new database has been created
3. Switch back to the grader user by running ```exit```
### Install git and clone the catalog project
* Run ```sudo apt install git```
* Create a directory in the ```/var/www/``` directory called item_catalog with ```sudo mkdir /var/www/item_catalog```
* ```cd``` to the 'item_catalog' directory, and clone the catalog project:
```sudo git clone https://github.com/scarrychris/full-stack-p2.git item_catalog```
Note: the "item_catalog" part at the end simply changes the directory name for the repository to 'item_catalog' instead of the default 'full-stack-p2'; this avoids problems later on as Apache does not like hyphens very much
* Change the ownership of the 'item_catalog' directory to ubuntu by running (while in /var/www):```sudo chown -R ubuntu:ubuntu item_catalog/```
* ```cd``` to the /var/www/item_catalog/item_catalog directory
* Change the name of the projectlogin.py file to ```__init__.py``` by running ```mv projectlogin.py __init__.py```
In ```__init__.py```, find line 362:
```
app.run(host='0.0.0.0', port=5000)
Change this line to:
app.run()
```
* Delete, rename, or move the ```database_setup.py``` file to another directory
Note: the default database for the Restaurant Menu Application is SQLite. The original database_setup.py file in the repository is configured for a SQLite database, and the database_setup_postgresql.py file is configured for PostgreSQL (only one change must be made to the file; see the "Switch the database in the application from SQLite to PostgreSQL" section below).
### Set up a vitual environment and install dependencies
* Start by installing pip (if it isn't installed already) with the following command:
```sudo apt install python-pip```
* Install virtualenv with ```apt``` by running ```sudo apt install python-virtualenv```
* ```cd``` to the ```/var/www/item_catalog/item_catalog/``` directory; choose a name for a temporary environment ('venv' is used in this example), and create this environment by running ```virtualenv venv``` (make sure to not use sudo here as it can cause problems later on)
* Activate the new environment, ```venv```, by running exactly```. venv/bin/activate```, you will know it's done when you see the following:
```
(venv) ubuntu@ip-172-26-8-80:/var/www/item_catalog/item_catalog$
```
* With the virtual environment active, install the following dependenies (note: with the exception of the ```libpq-dev``` package, make sure to not use sudo for any of the package installations as this will cause the packages to be installed globally rather than within the virtualenv):
```
pip install httplib2
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
pip install flask
sudo apt install libpq-dev (Note: this will install to the global evironment)
pip install psycopg2
```
* In order to make sure everything was installed correctly, run ```python __init__.py```; the following (among other things) should be returned:
```
* Serving Flask app "__init__" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
Deactivate the virtual environment by running deactivate
```
### Set up and enable a virtual host
* Create a file in ```/etc/apache2/sites-available/``` using ```touch item_catalog.conf```
* Add the following into the file:
```
<VirtualHost *:80>
		ServerName 35.169.159.50
		ServerAdmin scarrychris@gmail.com
		WSGIScriptAlias / /var/www/item_catalog/item_catalog.wsgi
		<Directory /var/www/item_catalog/item_catalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/item_catalog/item_catalog/static
		<Directory /var/www/item_catalog/item_catalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Note: the Options -Indexes lines ensure that listings for these directories in the browser is disabled.
* Run ```sudo a2ensite item_catalog``` to enable the virtual host
* The following prompt will be returned:
```
Enabling site item_catalog
To activate the new configuration, you need to run:
  systemctl reload apache2
  ```
* Run ```sudo systemctl reload apache2```
### Write a .wsgi file
* Apache serves Flask applications by using a ```.wsgi file```; create a file called```item_catalog.wsgi``` in ```/var/www/item_catalog```
* Add the following to the file:
```
activate_this = '/var/www/item_catalog/item_catalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/item_catalog/")
sys.path.insert(0,"/var/www/item_catalog/item_catalog")

from item_catalog import app as application
application.secret_key = '12345'
```
* Resart Apache: ```sudo systemctl restart apache2```
### Switch the database in the application from SQLite to PostgreSQL
* Replace line 25 in ```__init__.py```, line 63 in ```database_setup_postgresql.py```, and line 8 in ```populate.py``` with the following:
```engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')```
### Disable the default Apache site
* At some point during the configuration, the default Apache site will likely need to be disabled; to do this, run ```sudo a2dissite 000-default.conf```
* The following prompt will be returned:
```
Site 000-default disabled.
To activate the new configuration, you need to run:
  systemctl reload apache2
  ```
* Run ```sudo systemctl reload apache2```
### Change the ownership of the project direcotries
* Change the ownership of the project directories and files to the www-data user (this is done because Apache runs as the www-data user); while in the ```/var/www directory```, run:
```sudo chown -R www-data:www-data item_catalog/```
Note: if changes need to be made to the project files after the ownership of the directories has been switched to www-data, it is best to edit files as the www-data user; do this with the following command:
```sudo -u www-data vim INSERT_NAME_OF_FILE```
(Note: vim can be replaced here with nano or another text editor.)
### Set up the database schema and populate the database
* While in the ```/var/www/item_catalog/item_catalog/``` directory, activate the virtualenv by running ```. venv/bin/activate```
* Then run ```python database_setup_postgresql.py```
* Populate the database with ```python populate.py```
* Deactivate the virtualenv (run ```deactivate```)
* Resart Apache again: ```sudo systemctl restart apache2```
* Now open up a browser and check to make sure the app is working by going to``` http://35.169.159.50/```
### A few helpful commands to know
* Below is a list of commands that could be useful while setting up the server.
```lsb_release -a```
* Find out what version of Ubuntu is running
```whoami```
* Find out which user you are logged in as
```
>> import flask
>> flask.__version__
```
* Find out which version of Flask is installed (run this within Python)
```sudo systemctl restart apache2```
* Restart Apache (use this to make sure updates are reflected on the app)
```virtualenv --version```
* Find out which version of virtualenv is running
```which python```
* Find out where Python has been installed
(Note: this is especially useful when making sure that the virtualenv is working correctly; when the virtualenv is activated and set up correctly, running which python should not return ```/usr/bin/python``` but rather the file path to the directory where the virtualenv is located)
```vi + /var/log/apache2/error.log```
* View Apache error logs, and open the file starting with the last line (note: ```vi``` can be replaced here with nano or another text editor)
```sudo rm -rf INSERT_NAME_OF_VIRTUALENV_HERE```
* Delete a virtualenv and all of it's directories (be careful; this can't be undone)
```dropdb INSERT_NAME_OF_DATABASE```
* Drop (delete) a PostgreSQL database; this is helpful if changes are made to the ```database_setup_postgresql.py``` file; the current database should always be dropped and a new one created (run this command while logged in as the catalog user)
```sudo apachectl stop and sudo apachectl start```
* Stop and start Apache; stopping Apache breaks the database session, which makes it possible to drop a database
###  Some potentially useful information while configuring the server
#### Rebooting the virtual machine
* When logging in to the virtual machine, the following prompt may appear:
```*** System restart required ***```
* To restart the machine, simply run ```sudo reboot```
Note: it is important to keep in mind, that whenever the machine is rebooted, all information in RAM will disappear, the SSH session will end, and the machine will be offline for several minutes.
* After waiting a few minutes, SSH back into the machine as normal
#### Dropping and recreating a database
* At some point in the configuration, it may be necessary to drop the catalog database and recreate it. Here is one way to do that:
* Stop Apache by running ```sudo apachectl stop```
* Switch to the postgres user and enter psql: ```sudo -u postgres psql```
* Drop the current database (which is presumably called 'catalog'): ```drop database catalog;```
* Recreate the database: ```create database catalog owner catalog;```
* Exit PostgreSQL and psql (run ```exit```)
* Activate the virtual environmen:``` . venv/bin/activate```
* Run ```python populator.py```
* Deactivate the virtual environment (```deactivate```)
* Start Apache: ```sudo apachectl start```
#### Setting up SQLAlchemy logging
* At some point in the configuration, viewing logged SQLAlchemy information and errors may be userful. To do this, simply add the following into the imports section of the ```__init__.py``` file:
```
import logging
logging.basicConfig()
logging.getLogger('sqlalchemy.engine').setLevel(logging.DEBUG)
```
The SQLAlchemy logs will be added to the ```/var/log/apache2/error.log``` file.
### Fixing sudo: unable to resolve host ip-XX-XX-XX-XX error
* Open the ```/etc/hostname``` file (```/etc/hostname``` contains the name of the machine)
* Copy the file (it should say nothing more than ip-172-26-8-80, for example)
* Paste this into the first line of the /etc/hosts file, and add the following before it:
```localhost.localdomain```
* To be clear: the first line of the file should look like something this:
```127.0.0.1 localhost localhost.localdomain ip-172-26-8-80)```
* Run ```sudo hostname``` to make sure it worked; if it worked, something such as ```ip-172-26-8-80``` will be returned
### Fixing a servername error with Apache
When installing Apache, the following error may appear:
```
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message
```
* To fix this, open up the ```/etc/apache2/apache2.conf``` file
* Add in the following line at the end of the file:
```ServerName localhost```
* Restart Apache by running ```sudo systemctl restart apache2```
* If it did not work, the same error message will appear when restarting Apache
Note: this change will be overwritten when Apache is updated.

#### References
[Lohith94](https://github.com/Lohith94/linux-server-configuration)






