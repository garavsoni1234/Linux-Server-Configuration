
IP Address:
http://34.224.22.215/ (This address does not enable you to log into your Google account using Google OAuth 
http://34.224.22.215.nip.io/ (Use this address as it enables you to log in with you
google Oauth)
URL:https://github.com/garavsoni1234/Linux-Server-Configuration

Summary of software installed:
Ubuntu 16.04 (Downloaded through the windows store)



Summary of configuration made:
Step 1: Using Amazon Lightsail
Login to Lightsail and create an instance
Choose the Ubuntu 16.04 LTS platform
Picked the $3 per month option
Renamed the instance to "FinalProject"
Clicked Create
Step 2: Donwnload and install Ubuntu 16.04 on your windows machine from Microsoft Store
Step 3: Setting up SSH key
Open your Ubuntu terminal on your windows machine and type ssh-keygen
After you have generated the key, copy the SSH key by ssh-copy-id ubuntu@34.224.22.215 This will copy the contents to ~/.ssh/id_rsa.pub key and create a file called authorized_keys under ~/.ssh
Now go back to your lightsail terminal in the aws and enter the `/etc/ssh/sshd_config` command and change `PasswordAuthentication` to `yes`, then restart the sshd service by `sudo service sshd restart`, then set your password by `sudo passwd ubuntu`
Go back to your ubuntu 16.04 terminal on your machine. Now we have to copy the public key using SSH. For this, we will use the command -> cat ~/.ssh/id_rsa.pub | ssh ubuntu@34.224.22.215 "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys". When it asks `Are you sure you want to continue connecting (yes/no)?` select `yes`
Then copying Public Key Manually by `echo public_key_string >> ~/.ssh/authorized_keys`
Use this to set the permissions `chmod -R go= ~/.ssh` for `authorized_keys`
Use the chown -R ubuntu:ubuntu ~/.ssh to give the owner permissions to user
restart your terminal sudo systemctl restart ssh
Finally run this command ssh ubuntu@34.224.22.215 and you are in the remote server
Step 4: Update/Upgrade packages
sudo apt-get update
sudo apt-get upgrade
Step 5: Change from port 22 to port 2200
type in `sudo nano /etc/ssh/sshd_config`
and change port to 2200 (make sure it is not commented out.
save the changes and restart ssh `sudo service ssh restart`
Step 6: Now we can configure that firewall for security and allowance
Run the following commands:
sudo ufw status                 
sudo ufw default deny incoming 
sudo ufw default allow outgoing  
sudo ufw allow 2200/tcp          
sudo ufw allow www               
sudo ufw allow 123/udp           
sudo ufw deny 22 
Step 7: Edit Lightsail to realize the ports that were modified:
Go to instance and select your instance
Then, go into `Networking` and add the following 
		`Custom UDP 123`	
		`Custom TCP 2200`

Step 8: Add user grader and give it access:
As the ubuntu user add the user grader: `sudo adduser grader` and enter a new password
Give grader permission to sudo by using `sudo visudo`
Under `root ALL=(ALL:ALL) ALL`,`grader ALL=(ALL:ALL) ALL
On your localmachine run `ssh-keygen`
Give permissions chmod 700 .ssh and chmod 644.ssh/authorized_keys
In the /etc/ssh/sshd_config set Password Authenication to No
Restart SSH: sudo service ssh restart
On Local Machine run ssh -i ~/.ssh/grader_key -p 2200 grader@34.224.22.215

Step 9: Configure the local timezone to UTC
sudo dpkg-reconfigure tzdata
Then select your region
Step 10: Install Apache and mod-wsgi
Third-party resources:
As you are logged into `grader`, install Apache `sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3.
You need to install the Python3 mod_wsgi package `sudo apt-get install libapache2-mod-wsgi-py3`
Step 11: Install PostgresSQL
While you are logged into grade, install PostgreSQL:`sudo apt-get install postgresql`
Go into the PostgresSQL user and type sudo su - postgres
Open PostgresSQl terminal and type psql
Create a catalog user with a password
Then exit the postgres and psql
Create a user catalog: `sudo adduser catalog`
Run `sudo visudo`
add the following command `under `grader  ALL=(ALL:ALL) ALL`
Finaly create dabatebase catalog in psql
exit
Step 12: Install git
Make sure you are logged into grader
Go to the /var/www/catalog/ directory 
Type in `sudo git clone https://github.com/garavsoni1234/Item-Catalog catalog`
change the ownership of catalog by typing `sudo chown -R grader:grader catalog/`
cd into the catalog directory that has the project files
change the main file finalProject.py to __init__.py
sudo nano into the database_setup.py file and replace 
		`# engine = create_engine("sqlite:///catalog.db")` 			with
		`engine = 			create_engine('postgresql://catalog:PASSWORD@localhost/catal		og')`
Go to
Step 13: Authenicate login with Google
Go into Google Cloud Service: https://console.cloud.google.com/
Click `APIs & Services on left menu
Click on Credentials 
Create an OAuth Client ID of http://34.224.22.215.nip.io/ and http://34.224.22.215/
Download the client_secret.json from google and replace the file in the project
Copy the Client to the templates/login.html file 
Step 14: Install Virtual Environment
While logged into grader, install `pip3 sudo apt-get install python3-pip`
Install the virtual environment `sudo apt-get install python-virtualenv`
Go into catalog directory by `cd /var/www/catalog/catalog/`
Create the virtual environmet `sudo virtualenv -p python3 venv3`
change the ownership of grader `sudo chown -R grader:grader venv3/
Install the following dependencies:
		`pip install httplib2`
`pip install requests`
`pip install --upgrade oauth2client`
`pip install sqlalchemy`
`pip install flask`
`sudo apt-get install libpq-dev`
`pip install psycopg2`
Run the python3 __init__.py command
CTRL+C to deactivate
Step 16: Setup and enable virtual host
Go into the by `sudo nano /etc/apache2/mods-enabled/wsgi.conf`
		Add the following line:
		`WSGIPythonPath 
Add the following lines to /etc/apache2/sites-available/catalog.conf and add the following lines to configure the virtual host: `sudo nano /etc/apache2/sites-available/catalog.conf`
/var/www/catalog/catalog/venv3/lib/python3.5/site-packages`
`<VirtualHost *:80>
    ServerName 34.224.22.215
     ServerAdmin admin@34.224.22.215
    WSGIDaemonProcess itemcatalog python-path=/var/www \
        python-home=/var/www/catalog/venv
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
</VirtualHost>`
enable the host: `sudo a2ensite catalog`
Reload Apache: `sudo service apache2 reload`
Step 17: Set up the Flask application
Create /var/www/catalog/catalog.wsgi file add the following lines: by `sudo nano /var/www/catalog/catalog/catalog.wsgi`
	activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "..."

Issues to resolve:
Go into the main project file __init__.py and replace `xrange` to just `range`
replace line
line 77: `result = json.loads(h.request(url, 'GET')[1])` to 
`result = json.loads(h.request(url, "GET")[1].decode("utf-8"))`
