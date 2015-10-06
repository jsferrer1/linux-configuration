# P5 Project: Linux Server Configuration

## Perform Basic Configuration

### 1. Launch your Virtual Machine:  

* `https://www.udacity.com/account#!/development_environment`

1. Download Private Key  
2. Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.  
   `mv ~/Downloads/udacity_key.rsa ~/.ssh/`
3. Open your terminal and type in  
   `$ chmod 600 ~/.ssh/udacity_key.rsa`
4. In your terminal, type in:  
   `ssh -i ~/.ssh/udacity_key.rsa root@52.88.112.152`

Source: [Udacity][1]  
   
### 2. Create a new user named grader and grant this user sudo permissions.

#### 2.1 Create a new user and grant sudo permissions:  

1. Create a new user:  
   `$ sudo adduser grader`
2. Create a `sudoers.d` file for grader:  
   `$ touch /etc/sudoers.d/grader`
3. Edit the file:  
   `$ sudo nano /etc/sudoers.d/grader`
4. Include the entry below into the grader file:  
   `grader ALL=(ALL) NOPASSWD:ALL`

Source: [DigitalOcean][2]  
   
#### 2.2 Enable key-based authentication:  

1. Sudo as the new user:  
   `$ su - grader`
1. Make an `.ssh` folder:  
   `$ mkdir .ssh`
2. Create an `authorized_keys` file:  
   `$ touch .ssh/authorized_keys`
4. Edit the file:  
  `$ nano .ssh/authorized_keys`
5. Copy & paste the content of local `ssh_rsa_key.pub` into the `authorized_keys`
6. Change directory permissions:  
   `$ chmod 0700 .ssh`
7. Change file permissions:  
  `$ chmod 0600 .ssh/authorized_keys`

Source: [DigitalOcean][2.2]  
 
### 3. Update all currently installed packages.

   1. Update all the packages:  
      `$ sudo apt-get update`
   2. Upgrade all the packages:  
      `$ sudo apt-get upgrade`

Source: [Ask Ubuntu][3]  

### 4. Configure the local timezone to UTC.

1. Reconfigure `tzdata` using:  
   `$ sudo dpkg-reconfigure tzdata`
2. Then chose `None of the above`, then `UTC`. 
3. Install the ntp daemon ntpd for regular and improving time sync:  
  `$ sudo apt-get install ntp`
4. Chose closer NTP time servers:  
  1. Open the NTP configuration file:  
    `$ sudo nano /etc/ntp.conf`
  2. Add the Asia pool zone:
     ```
     server 0.asia.pool.ntp.org
     server 1.asia.pool.ntp.org
     server 2.asia.pool.ntp.org
     server 3.asia.pool.ntp.org  
     ```

Source: [Ubuntu documentation][4]
     
### 5. Change the SSH port from 22 to 2200   

1. Edit the ssh config file:  
   `$ sudo nano /etc/ssh/sshd_config`
2. Change the `Port` from `22` to `2200`  
3. Change `PermitRootLogin` from `without-password` to `no`.  
4. Change `PasswordAuthentication` from `yes` to `no`.  
5. Save the ssh config file then restart the service:   
   `$ sudo service ssh restart`
6. Exit the server:  
   `$ exit`   
7. Login using the ssh key:  
   `cmd> ssh grader@52.88.112.152 -p 2200 -i .ssh/ssh_rsa_key`  
   (then enter the passphrase)
8. Include the ip address into the `/etc/hosts` file:  
   `127.0.0.1 ip-10-20-10-194`   
   
Source: [Ask Ubuntu][5]  
   
### 6. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)   
   
1. (Optional) Check first the firewall status:  
   `$ sudo ufw status verbose`
2. Deny all incoming packets:  
   `$ sudo ufw default deny incoming`
3. Allow all outgoing packets:  
   `$ sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 (SSH):  
   `$ sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 (HTTP):  
   `$ sudo ufw allow 80/tcp`
6. Allow incoming TCP packets on port 123 (NTP):  
   `$ sudo ufw allow 123/udp`
7. Enable the ports now:  
   `$ sudo ufw enable`
8. Reboot the server:  
   `$ sudo shutdown -r now`  

Source: [Ubuntu documentation][6]  
   
## Install your application
 
### 7. Install and configure Apache to serve a Python mod_wsgi application

   1. Install Apache web server:  
      `$ sudo apt-get install apache2`
   2. Install **`mod_wsgi`** for serving Python apps:  
      `$ sudo apt-get install libapache2-mod-wsgi python-dev`  
   3. Enable mod_wsgi (if not already enabled):  
      `$ sudo a2enmod wsgi`  

Source: [Udacity][7]
   
### 8. Install and configure PostgreSQL:
Source: [DigitalOcean][8]


#### 8.1. Do not allow remote connections
  
1. Install `postgresql`:  
   `$ sudo apt-get install postgresql postgresql-contrib`
2. Open the configuration file:  
   `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf`
3. Ensure that the config file contains the following entries (comments included):  
  ```
  # TYPE  DATABASE        USER            ADDRESS                 METHOD
  # Database administrative login by Unix domain socket
  local   all             postgres                                peer
  # "local" is for Unix domain socket connections only
  local   all             all                                     peer
  # IPv4 local connections:
  host    all             all             127.0.0.1/32            md5
  # IPv6 local connections:
  host    all             all             ::1/128                 md5        
  ```

#### 8.2. Create a new user named catalog that has limited permissions to your catalog application database
    
1. Launch `psql` as `postgres` user:  
   `$ sudo su - postgres`, then `$ psql` 
2. Create the catalog user:  
   `# CREATE USER catalog WITH PASSWORD 'catalog';` (`#` is the psql prompt)
3. Allow the user to create database tables:  
   `# ALTER USER catalog CREATEDB;`
4. Create the catalog database:  
   `# CREATE DATABASE catalog WITH OWNER catalog;`
5. Connect to the catalog database as postgres:  
   `# \c catalog`
6. Revoke all rights:  
  `# REVOKE ALL ON SCHEMA public FROM public;`
7. Grant only access to the catalog role:  
  `# GRANT ALL ON SCHEMA public TO catalog;`
8. Exit out of PostgreSQl and the postgres user:  
  `# \q` 

   
#### 8.3. Create a role "www-data" for apache purposes:   

1. Launch `psql` as `postgres` user:  
   `$ sudo su - postgres`, then `$ psql` 
2. Create the role with password, login, and createdb privileges:  
   `# create role "www-data" with password 'www-data';`  
   `# alter role "www-data" with login;`
   `# alter role "www-data" with createdb;`
3. Exit out of PostgreSQl and the postgres user:  
  `# \q`, then `$ exit` 
        
### 9. Install git, clone and set up your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

#### 9.1 Install git & clone

1. Install git:  
   `$ sudo apt-get install git`
2. Install required packages:  
   `$ sudo apt-get install python-pip`  
   `$ sudo pip install sqlalchemy`
   `$ sudo apt-get install python-psycopg2`  
3. Setup the catalog app in `/var/www`:  
   `$ cd /var/www`
4. Clone the catalog app then change permissions:  
   `$ sudo git clone https://github.com/jsferrer1/catalog-app.git catalog`  
   `$ sudo adduser catalog` then give `sudo` permission to `catalog`  
   `$ sudo chown -R catalog:catalog catalog`  
   `$ sudo chmod 777 catalog/catalog/static/img`
5. Create the catalog database schema then load sample data:  
   `cd catalog/catalog`
   `$ python database_setup.py`  
   `$ python loaditems.py`
6. Rename the main application program:  
   `$ mv application.py __init__.py`

Source: [GitHub][9.1]

#### 9.2 Deny access to .git directory:  

1. Go to the `catalog` folder:  
   `$ cd ..`
2. Rename the htaccess file:  
   `$ mv htaccess .htaccess`  
   The `.htaccess` file should contain `RedirectMatch 404 /\.git`

Source: [Stackoverflow][9.2]

#### 9.3 Install required modules & packages:  
1. Install virtualenv:  
  `$ sudo pip install virtualenv`
2. Set virtual environment to name 'venv':  
  `$ sudo virtualenv venv`
3. Enable all permissions for the new virtual environment (no sudo should be used within):  
  `$ sudo chmod -R 777 venv`
4. Activate virtual environment:   
  `$ source venv/bin/activate`
5. Install httplib2 module in venv:  
  `$ pip install httplib2`
6. Install requests module in venv:  
  `$ pip install requests`
7. Install flask.ext.seasurf (only seems to work when installed globally):  
  `$ sudo pip install flask-seasurf`
8. Install oauth2client.client:  
  `$ sudo pip install --upgrade oauth2client`
9. Install Dictionary to XML:  
   `$ sudo pip install dicttoxml`
10. Install File Security:  
   `$ sudo pip install werkzeug==0.8.3`
11. Install Flask Web Framework:  
   `$ sudo pip install flask==0.9`
12. Install Flask Login:  
   `$ sudo pip install Flask-Login==0.1.3`
13. Deactivate the environment:  
   `$ deactivate`

Source: [Stackoverflow][9.3]              

### 10. Test the site

1. Move the catalog configuration file to the `sites-available`:  
   - `$ mv /var/www/catalog/catalog.conf /etc/apache2/sites-available/`
   - Edit then add the following `ServerAlias` entry:  
     `sudo nano /etc/apache2/sites-available/catalog.conf`
     `ServerAlias ec2-52-88-112-152.us-west-2.compute.amazonaws.com`
2. Ensure that the `.wsgi` web interface file exists in `/var/www/catalog`:   
   `$ ls /var/www/catalog/catalog.wsgi`  
3. Modify the client secrets file to include authentication information:  
   `client_secrets.json` : include client_id and client_secret information  
   `fb_client_secrets.json` : include the app_id and app_secret information 
4. Disable the default site:  
   `$ sudo a2dissite 000-default`
5. Enable the catalog site:  
   `$ sudo a2ensite catalog`
6. Restart Apache:  
   `$ sudo service apache2 restart`
7. Open a browser and put in your public ip-address as url, e.g. 52.88.112.152 
   - if everything works, the application should come up
   - http://ec2-52-88-112-152.us-west-2.compute.amazonaws.com/
    
Source: [Udacity][10.1] and [Apache][10.2]  

## Additional Functionality

### 11. Security

#### 11.1 Configure Firewall to monitor for repeated unsuccessful login attempts and ban attackers

1. Install Fail2ban:  
  `$ sudo apt-get install fail2ban sendmail`
2. Copy the default config file:  
  `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
3. Check and change the default parameters:  
    1. Open the local config file:  
      `$ sudo nano /etc/fail2ban/jail.local`
    2. Set the following Parameters:  
       ```  
       set bantime  = 1800  
       destemail = root@localhost  
       action = %(action_mwl)s  
       under [ssh] change port = 2200
       ```  
4. Stop the service:  
  `$ sudo service fail2ban stop`
5. Start it again:  
  `$ sudo service fail2ban start`

Source: [DigitalOcean][11.1]  
  
### 11.2 Install cron scripts to automatically manage package updates

1. Install the unattended-upgrades package:  
  `$ sudo apt-get install unattended-upgrades`
2. Enable the unattended-upgrades package:  
  `$ sudo dpkg-reconfigure -plow unattended-upgrades`  
  then select `<Yes>`

Source: [Ubuntu documentation][11.2]  

### 12. Server Monitoring

1. Install monit  
   `$ sudo apt-get install -y monit`  
2. Configure monit  
   ```
   $ sudo nano /etc/monit/conf.d/monit.conf
   # update email alert
     set alert root@localhost
   # setup mail server
     set mailserver localhost                   # fallback relay
                    
   # uncomment or update servieces
   # setup http deamon
     set httpd port 2812
       use address localhost  # only accept connection from localhost
       allow 0.0.0.0/0.0.0.0        # allow localhost to connect to the server and
       allow admin:monit      # require user 'admin' with password 'monit'
   # set basic system monitor
     check system localhost
       if loadavg (1min) > 4 then alert
       if loadavg (5min) > 2 then alert
       if memory usage > 75% then alert
       if swap usage > 25% then alert
       if cpu usage (user) > 70% then alert
       if cpu usage (system) > 30% then alert
       if cpu usage (wait) > 20% then alert
   # setup for apache monitor
     check process apache2 with pidfile /var/run/apache2/apache2.pid
       start program = "/etc/init.d/apache2 start" with timeout 60 seconds
       stop program  = "/etc/init.d/apache2 stop"
       if cpu > 60% for 2 cycles then alert
       if cpu > 80% for 5 cycles then restart
       if totalmem > 200.0 MB for 5 cycles then restart
       if children > 250 then restart
       if loadavg(5min) greater than 10 for 8 cycles then stop
       if failed host localhost port 80
             protocol apache-status  dnslimit > 25% or 
                                     loglimit > 80% or 
                                     waitlimit < 20%
             then alert
       group server
   ```
3. Check control file syntax and start monit  
   `$ sudo monit -t`  
   `$ sudo monit reload`  
   `$ sudo monit start all`  

Source: [DigitalOcean][12]  
   
## Resources

[1]: https://www.udacity.com/account#!/development_environment "Instructions for SSH access to the instance"
[2]: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps "How To Add and Delete Users on an Ubuntu 14.04 VPS"
[2.2]: https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server "How To Configure SSH Key-Based Authentication on a Linux Server"
[3]: http://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade "What is the difference between apt-get update and upgrade?"      
[4]: https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29 "Ubuntu Time Management"
[5]: http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server "Create a new SSH user on Ubuntu Server"
[6]: https://help.ubuntu.com/community/UFW "UFW - Uncomplicated Firewall"
[7]: http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html "A Step by Step Guide to Install LAMP (Linux, Apache, MySQL, Python) on Ubuntu"
[8]: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps "How To Secure PostgreSQL on an Ubuntu VPS"  
[9.1]: https://help.github.com/articles/set-up-git/#platform-linux "Set Up Git for Linux"   
[9.2]: http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible "Make .git directory web inaccessible"   
[9.3]: http://stackoverflow.com/questions/14695278/python-packages-not-installing-in-virtualenv-using-pip "python packages not installing in virtualenv using pip"
[10.1]: http://discussions.udacity.com/t/oauth-provider-callback-uris/20460 "OAuth Provider callback uris"
[10.2]: http://httpd.apache.org/docs/2.2/en/vhosts/name-based.html "Name-based Virtual Host Support"
[11.1]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04 "How To Install and Use Fail2ban on Ubuntu 14.04"
[11.2]: https://help.ubuntu.com/community/AutomaticSecurityUpdates "AutomaticSecurityUpdates"  
[12]: https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-monit "How To Install and Configure Monit"  
