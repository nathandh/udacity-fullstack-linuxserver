# udacity-fullstack-linuxserver
**Linux Server Configuration Project** as part of Udacity's FullStack NanoDegree

## Implementation:
#### AWS LightSail IP instance IP address:
* 54.211.234.105
#### SSH Port:
* 2200
#### Hosted Flask Catalog Item App:
* http://54.211.234.105/
#### Server Setup / Software Installed:
1. Installed Tmux for setup ease, and created 'Udacity' session:
	* ``` sudo apt-get install tmux ```
	* ``` tmux new-session -s "Udacity" ```
2. Ensured VIM was installed for editing:
	* ``` sudo apt-get install vim ```
3. Updated All Packages on System
	* ``` sudo apt-get update && sudo apt-get upgrade
4. Added a 'grader' user, 'developers' group, and added 'grader' to the group:
	* ``` sudo adduser grader ```
	* ``` sudo groupadd developers ```
	* ``` sudo usermod -a -G developers grader ```
5. Edited 'sudoers' list to include grader
	* ``` sudo vim /etc/sudoers.d/grader ```
		1. Inserted the following lines in the new file:
			* ``` # User rules for Grader 
				  grader ALL=(ALL) NOPASSWD:ALL
			  ```
6. Installed 'finger' and checked 'grader' info
	* ``` sudo apt-get install finger ```
	* ``` finger grader ```
7. Prepared for .ssh login only for 'grader'
	* ``` mkdir ~/.ssh ```
	* ``` touch ~/.ssh/authorized_keys ```
	* ``` chmod 700 ~/.ssh ```
	* ``` chmod 644 ~/.ssh/authorized_keys ```
8. Edited '/etc/ssh/sshd_config':
	* - to not allow password login, and force key login:
		1. ``` PasswordAuthentication no ```
	* - to allow for Public Key authentication:
		1. ``` PubkeyAuthentication yes ```
	* - to change PORT to 2200
		1. ``` port 2200 ```
9. Configured UFW firewall for server security:
	* ``` sudo ufw default deny incoming ```
	* ``` sudo ufw default allow outgoing ```
	* ``` sudo ufw allow 2200/tcp ```
	* ``` sudo ufw allow www ```
	* ``` sudo ufw allow ntp ```
	* ``` sudo ufw enable ```
	* ``` sudo ufw status ```
10. Restarted SSHD:
	* ``` sudo systemctl restart sshd ```
11. Ensured timezone was set to UTC:
	* ``` sudo dpkg-reconfigure tzdata ```
12. Created ssh private/public key pair on my non-remote machine:
	* ``` ssh-keygen ```
13. Pasted generated 'public' key in my 'authorized_keys' file for 'grader'
	* ``` vim ~/.ssh/authorized_keys ```
	* Pasted public key as 1st line
14. Ensured AWS Lightsail restrictions allowed for port 2200 as well in console
15. Logged on via my private/public key pair on port 2200 as 'grader' user
16. Setup for Hosted Web Catalog Item APP:
	* Installed Apache2 web server
		* ``` sudo apt-get install apache2 ```
	* Installed Mod-WSGI Apache module:
		* ``` sudo apt-get install libapache2-mod-wsgi ```
	* Restarted Apache and tested that base intstall was success:
		* ``` sudo systemctl restart apache2 ```
		* Browsed to: http://54.211.234.105/ to see default Apache page.
	* Installed PostgreSQL:
		* ``` sudo apt-get install postgresql ```
	* Made a directory under '/opt' to house my Catalog web app:
		* ``` mkdir -p /opt/webapps ```
	* Cloned my Catalog app into webapps directory:
		* ``` cs /opt/webapps/ ```
		* ``` sudo git clone https://github.com/nathandh/udacity-fullstack-catalog-postgres.git ```
		* ``` sudo chown grader:developers /opt/webapps/udacity-fullstack-catalog-postgres ```
	* Made some small changes to 'catalog.py', 'database_setup.py', and 'test_data.py' to get app running:
		* These changes are tracked in a new branch 'aws_devel' of my Postgres Catalog App:
			* ``` https://github.com/nathandh/udacity-fullstack-catalog-postgres.git ```
	* Created a Apache /var/www level folder to house the *.wsgi script:
		* ``` sudo mkdir -d /var/www/udacity_catalog ```
		* Created a 'catalog.wsgi' script in this new folder:
			* ``` sudo touch var/www/udacity_catalog ```
		* Edited 'catalog.wsgi' to have the following:
			* ``` import sys
			      import logging
				  logging.basicConfig(stream=sys.stderr)
				  sys.path.insert(0, '/opt/webapps/udacity-fullstack-catalog-postgres')

				  from catalog import app as application
				  application.secret_key = 'secret'
			  ```
	* Created a 'udacity_catalog.conf' file in Apache2 'sited-available' folder:
		* ``` sudo vim /etc/apache2/sites-available/udacity_catalog.conf ```
	* Edited 'udacity_catalog.conf' appropriately:
		* ``` <VirtualHost *>
					ServerName udacity.catalog.local
					ServerAdmin webmaster@localhost

					DocumentRoot /opt/webapps/udacity-fullstack-catalog-postgres

					ErrorLog ${APACHE_LOG_DIR}/error.log
					CustomLog $(APACHE_LOG_DIR}/access.log combined
					# LogLevel info

					# WSGI specific
					WSGIScriptAlias / /var/www/udacity_catalog/catalog.wsgi

					<Directory /opt/webapps/udacity-fullstack-catalog-postgres>
						Order allow,deny
						Allow from all
					</Directory>

					<Directory /var/www/udacity_catalog>
						Order allow,deny
						Allow from all
					</Directory>
				</VirtualHost>
			```
	* Edited '/etc/hosts' to allow for our app to run
		* ``` sudo vim /etc/hosts ```
			* Added 2 lines specific for our app URI handling
				1. ``` 127.0.0.1 udacity.catalog.local ```
				2. ``` 127.0.0.1 udacity-catalog.com ```
	* Created a symbolic link from 'sited-available' to sited-enabled'
		* ``` sudo ln -s /etc/apache2/sites-available/udacity_catalog.conf /etc/apache2/sites-enabled/ ```
	* Restarted apache2 so changes could take effect
		* ``` sudo systemctl restart apache2 ```
	* Installed Extra Needed libraries and applications:
		* ``` sudo apt-get install python-pip
			  sudo apt-get install python-flask
			  sudo apt-get install python-flask-sqlalchemy
			  sudo apt-get install python-psycopg2
			  sudo apt-get install python-httplib2
			  sudo apt-get install python-requests
			  sudo apt-get install python-simplejson
			  sudo -H pip install oauth2client
		  ```
	* Went to Google Developer Console and Edited Client_Secrets to include new IP address
	* Replaced 'clien_secrets.json' contents with new Auth info from Google
	* Logged on as 'postgres' user and ran 'udacity_catalog.sql' to create DB in PostgreSQL:
		* ``` sudo -u postgres -i ```
		* ``` psql ```
		* ``` \i udacity_catalog.sql ```
		* Created a 'catalog' limited permission catalog user:
			* ``` CREATE USER catalog  WITH NOSUPERUSER NOCREATEDB NOCREATEROLE; ```
		* Altered user to include a password:
			* ``` ALTER USER "catalog" WITH password "<my_chosen_password>"; ```
	* Ensured 'database_setup.py' connection string included 'catalog' user information:
		* ``` vim database_setup.py ```
	* Final Checks, and restarting of server for Updated that required reboot:
		* Check 'ufw' rules:
			* ``` sudo ufw status ```
		* Restart server:
			* ``` sudo shutdown -r now ```
	* After reboot, check that Hosted Catalog App runs as expected:
		* http://54.211.234.105/
#### Referenced third-party resources:
	* http://docs.sqlalchemy.org/en/latest/dialects/postgresql.html
	* http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
	* Numeours StackOverflow questions/answers relating to issues with Flask, Mod_Wsgi, Apache2
#### License:
Licensed for use unde the MIT License
