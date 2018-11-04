#Linux Server Configuration

##About
This is the third project for Udacity's Full Stack Web Developer Nanodegree - Term 2.

This page explains how to securely set up a Linux distribution on a virtual machine, install and configure a web and database server to host a web application on Amazon Web Services.

- The Linux distribution is Ubuntu 16.04 LTS.
- The virtual private server is Amazon Lighsail.
- The web application is my Item Catalog project created earlier in this Nanodegree program.
- The database server is PostgreSQL.
- My local machine is on Windows 10.
- You can visit http://18.197.51.8.xip.io for the website deployed.

## Technical Information About the Project

- **Server IP Address:** 18.197.51.8
- **Server Alias:** 18.197.51.8.xip.io
- **SSH server access port:** 2200
- **SSH login username:** grader
- **SSH login password:** grader
- **Application URL:** http://18.197.51.8.xip.io

## Summary of software used

- Got all system packages updates available: sudo apt-get update
- Installed all system packages updates: sudo apt-get upgrade
- Python installed
- Flask installed
- Apache installed.
- Git installed.
- WSGI_mod installed

## Set up a server

### Start a new Ubuntu Linux server instance on Amazon Lightsail 

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
- Once you are login into the site, click `Create instance`. 
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Choose a instance plan (I took the cheapest, $5/month which was free for 1st month).
- Keep the default name provided by AWS or rename your instance.
- Click the `Create` button to create the instance.
- Wait for the instance to start up.


### SSH into the server

- From the `Account` menu on Amazon Lightsail, click on `SSH keys` tab and download the Default Private Key.
- Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `lightsail_key.rsa`.
- In your terminal, type: `chmod 600 ~/.ssh/lightsail_key.rsa`.
- To connect to the instance via the terminal: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.197.51.8`, 
  where `18.197.51.8` is the public IP address of the instance.

## Secure the server

### Update and upgrade installed packages

```
sudo apt-get update
sudo apt-get upgrade
```
### Change the SSH port from 22 to 2200

- Edit the `/etc/ssh/sshd_config` file: `sudo nano /etc/ssh/sshd_config`.
- Change the port number on line 5 from `22` to `2200`.
- Save and exit using CTRL+X and confirm with Y.
- Restart SSH: `sudo service ssh restart`.

### Configure the Uncomplicated Firewall (UFW)

- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  ```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  ```

- Turn UFW on: `sudo ufw enable`. The output should be like this:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```

- Check the status of UFW to list current roles: `sudo ufw status`. The output should be like this:

  ```
  Status: active
  
  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                                
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  ```

- Exit the SSH connection: `exit`.

- Click on the `Manage` option of the Amazon Lightsail Instance, 
then the `Networking` tab, and then change the firewall configuration to match the internal firewall settings above.


- Allow ports 80(TCP), 123(UDP), and 2200(TCP).

- From your local terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@13.59.39.163`, where `13.59.39.163` is the public IP address of the instance.

## Give `grader` access


### Create a new user account named `grader`

- While logged in as `ubuntu`, add user: `sudo adduser grader`. 
- Enter a password (twice) and fill out information for this new user.


### Give `grader` the permission to sudo

- Edits the sudoers file: `sudo visudo`.
- Search for the line that looks like this:
  ```
  root    ALL=(ALL:ALL) ALL
  ```

- Below this line, add a new line to give sudo privileges to `grader` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y.

### Create an SSH key pair for `grader` using the `ssh-keygen` tool

- On the local machine:
  - Run `ssh-keygen`
  - Enter file in which to save the key (I gave the name `grader_key`) in the local directory `~/.ssh`
  - Enter in a passphrase twice. Two files will be generated (  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
  - Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file
  - Log in to the grader's virtual machine
- On the grader's virtual machine:
  - Create a new directory called `~/.ssh` (`mkdir .ssh`)
  - Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
  - Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - Check in `/etc/ssh/sshd_config` file if `PasswordAuthentication` is set to `no`
  - Restart SSH: `sudo service ssh restart`
- On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@18.197.51.8`.

## Prepare to deploy the project

### Configure the local timezone to UTC

- While logged in as `grader`, configure the time zone: `sudo dpkg-reconfigure tzdata`. You should see something like that:

  ```
  Current default time zone: 'UTC'
  Local time is now:      Fri Oct 20 01:55:16 UTC 2018.
  Universal Time is now:  Fri Oct 20 01:55:16 UTC 2018.
  ```

###  Install and configure Apache to serve a Python mod_wsgi application

- While logged in as `grader`, install Apache: `sudo apt-get install apache2`.
- Enter public IP of the Amazon Lightsail instance into browser. If Apache is working you could see the page stating so.

- My project is built with Python 3. So, I need to install the Python 3 mod_wsgi package:  
 `sudo apt-get install libapache2-mod-wsgi-py3`.
- Enable `mod_wsgi` using: `sudo a2enmod wsgi`.

###  Install and configure PostgreSQL

- While logged in as `grader`, install PostgreSQL:
 `sudo apt-get install postgresql`.
- PostgreSQL should not allow remote connections. In the  `/etc/postgresql/9.5/main/pg_hba.conf` file, you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```

- Switch to the `postgres` user: `sudo su - postgres`.
- Open PostgreSQL interactive terminal with `psql`.
- Create the `catalog` user with a password and give them the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```

- List the existing roles: `\du`. The output should be like this:
  ```
                                     List of roles
   Role name |                         Attributes                         | Member of 
  -----------+------------------------------------------------------------+-----------
   catalog   | Create DB                                                  | {}
   postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
  ```

- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.
- Create a new Linux user called `catalog`: `sudo adduser catalog`. Enter password and fill out information.
- Give to `catalog` user the permission to sudo. Run: `sudo visudo`.
- Search for the lines that looks like this:
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Below this line, add a new line to give sudo privileges to `catalog` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  catalog  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y.
- Verify that `catalog` has sudo permissions. Run `su - catalog`, enter the password, run `sudo -l` and enter the password again. The output should be like this:

  ```
  Matching Defaults entries for catalog on ip-172-26-13-170.us-east-2.compute.internal:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User catalog may run the following commands on ip-172-26-13-170.us-east-2.compute.internal:
      (ALL : ALL) ALL
  ```

- While logged in as `catalog`, create a database: `createdb catalog`.
- Run `psql` and then run `\l` to see that the new database has been created. The output should be like this:
  ```
                                    List of databases
     Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
  -----------+----------+----------+-------------+-------------+-----------------------
   catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
   postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
   template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
   template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
  (4 rows)
  ```
- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.

###  Install git

- While logged in as `grader`, install `git`: `sudo apt-get install git`.

## Deploy the Item Catalog project

### Clone and setup the Item Catalog project from the GitHub repository 

- While logged in as `grader`, create `/var/www/catalog/` directory.
- Change to that directory and clone the catalog project:<br>
`sudo git clone https://github.com/abhay-jain/item-catalog catalog`.
- From the `/var/www` directory, change the ownership of the `catalog` directory to `grader` using: `sudo chown -R grader:grader catalog/`.
- Change to the `/var/www/catalog/catalog` directory.
- Rename the `application.py` file to `__init__.py` using: `mv application.py __init__.py`.

- In `__init__.py`, replace line 27:
  ```
  # app.run(host="0.0.0.0", port=8000, debug=True)
  app.run()
  ```

- In `database.py`, replace line 9:
   ```
   # engine = create_engine("sqlite:///catalog.db")
   engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
   ``` 

### Authenticate login through Google

- Go to [Google Cloud Plateform](https://console.cloud.google.com/).
- Click `APIs & services` on left menu.
- Click `Credentials`.
- Create an OAuth Client ID (under the Credentials tab), and add http://18.197.51.8.xip.io as authorized JavaScript 
origin.
- Add http://18.197.51.8.xip.io/login as authorized redirect URI.
- Download the corresponding JSON file, open it et copy the contents.
- Open `/var/www/catalog/catalog/client_secret.json` and paste the previous contents into the this file.
- Replace the client ID to line 25 of the `templates/login.html` file in the project directory.

### Install the virtual environment and dependencies

- While logged in as `grader`, install pip: `sudo apt-get install python3-pip`.
- Install the virtual environment: `sudo apt-get install python-virtualenv`
- Change to the `/var/www/catalog/catalog/` directory.
- Create the virtual environment: `sudo virtualenv -p python3 venv3`.
- Change the ownership to `grader` with: `sudo chown -R grader:grader venv3/`.
- Activate the new environment: `. venv3/bin/activate`.
- Install the following dependencies:
  ```
  pip install httplib2
  pip install requests
  pip install --upgrade oauth2client
  pip install sqlalchemy
  pip install flask
  sudo apt-get install libpq-dev
  pip install psycopg2
  ```

- Deactivate the virtual environment: `deactivate`.

### Set up and enable a virtual host

- Add the following line in `/etc/apache2/mods-enabled/wsgi.conf` file 
to use Python 3.

  ```
  #WSGIPythonPath directory|directory-1:directory-2:...
  WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages
  ```

- Create `/etc/apache2/sites-available/catalog.conf` and add the 
following lines to configure the virtual host:

  ```
  <VirtualHost *:80>
	  ServerName 18.197.51.8
    ServerAlias ec2-18-197-51-8.eu-central-1.compute.amazonaws.com
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

- Enable virtual host: `sudo a2ensite catalog`. The following prompt will be returned:
  ```
  Enabling site catalog.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

- Reload Apache: `sudo service apache2 reload`.


### Set up the Flask application

- Create `/var/www/catalog/catalog.wsgi` file add the following lines:

  ```
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
  ```

- Restart Apache: `sudo service apache2 restart`.

### Disable the default Apache site

- Disable the default Apache site: `sudo a2dissite 000-default.conf`. 
The following prompt will be returned:

  ```
  Site 000-default disabled.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

- Reload Apache: `sudo service apache2 reload`.

### Launch the Web Application

- Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
- Restart Apache again: `sudo service apache2 restart`.
- Open your browser to http://18.197.51.8 .

##References

- ServerPilot, [How to Create a Server on Amazon Lightsail](https://serverpilot.io/community/articles/how-to-create-a-server-on-amazon-lightsail.html).
- Official Ubuntu Documentation, [UFW - Uncomplicated Firewall](https://help.ubuntu.com/community/UFW).
- TechRepublic, [How to install and use Uncomplicated Firewall in Ubuntu](https://www.techrepublic.com/article/how-to-install-and-use-uncomplicated-firewall-in-ubuntu/).
- DigitalOcean, [How To Protect SSH with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04).
- Official Ubuntu Documentation, [Automatic Updates](https://help.ubuntu.com/lts/serverguide/automatic-updates.html).
- Ubuntu Wiki, [AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates).
- DigitalOcean, [Updating Ubuntu 14.04 -- Security Updates](https://www.digitalocean.com/community/questions/updating-ubuntu-14-04-security-updates).
- DigitalOcean, [How To Set Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2).
- Ubuntu Wiki, [SSH/OpenSSH/Keys](https://help.ubuntu.com/community/SSH/OpenSSH/Keys).
- Ubuntu Wiki, [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)
- Ask Ubuntu, [How do I change my timezone to UTC/GMT?](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)
- DigitalOcean, [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).
- Flask documentation, [virtualenv](http://flask.pocoo.org/docs/0.12/installation/).
- [Create a Python 3 virtual environment](https://superuser.com/questions/1039369/create-a-python-3-virtual-environment).