# Project: Linux Server Configuration
The third project for Udacity Full Stack Developer Nanodegree.
### IP Address and SSH Port
- IP: http://18.195.218.241.xip.io/
- Port: 2200

Log in as **grader** on:

```
$ssh -i ~/.ssh/FSD_Udacity  grader@18.195.218.241 -p 2200
```

# Server Security
### Update & Upgrade Currently Installed Packages
```
# Update available packages
$sudo apt-get update

# Upgrading installed packages
$sudo apt-get upgrade

# Automatically remove rot required packages
$sudo apt-get autoremove
```
### Firewall Configuration
**First, Make sure to configure the Lightsail instance's firewall to allow the following changes**
- Go to Lightsail instance **networking** settings > Firewall > Add a new rule.
- Using Terminal:

```
# Check the status of the firewall and make sure that it is inactive
$sudo ufw default deny incoming
$sudo ufw default allow outgoing
$sudo ufw allow ssh allow 2200/tcp
$sudo ufw allow www
$sudo ufw allow ntp
$sudo ufw enable
```
### New User: Grader
- Add user

```
$sudo adduser grader
```
- Give **grader** the permission to **$sudo**

```
# With user Ubuntu
$sudo cp /etc/$sudoers.d/ubuntu /etc/$sudoers.d/grader

# Edit grader file
$sudo nano /etc/$sudoers.d/grader
# Change 'ubuntu' to 'grader'
```
### Generating Key Pairs
- On your local machine run the following command to generate the key pairs

```
$ssh-keygen
# Enter file in which to save the key, example:
/Users/Udacity/.ssh/linuxCourse
```
You will end up having two files, one for the private key and the other for the public key.

- Login to your instance as grader, and run the following

```
$cd .ssh
$sudo nano authorized_keys
# On a new line, paste the public key file content, then save the file

$chmod 700 .ssh
$chmod 644 .ssh/authorized_keys
```
- Login to your instance using your new key:

```
$ssh -i ~/.ssh/FSD_Udacity  grader@18.195.218.241 -p 2200
```
### Disable Remote Login for Root User & Changing Port From 22 to 2200
```
$sudo nano /etc/ssh/sshd_config
```
- Change port from 22 to 2200
- Change PermitRootLogin to **no**

# Prepare to Deploy the Item Catalog Project
### Configure Local Timezone to UTC
```
$sudo dpkg-reconfigure tzdata
# Choose None of the above
# Choose UTC
```
### Install and Configure Apache to Serve a Python Mod_WSGI Application
**For applications developed using Python3**
```
$sudo apt-get install libapache2-mod-wsgi-py3
```
### Install and Configure PostgreSQL
```
$sudo apt-get install postgresql
$sudo -u postgres createuser --interactive
```
### Install git
```
$sudo apt install git
$cd /var/www
$sudo git clone https://github.com/nuhaFS/ItemCatalog-FSDN.git
$sudo chown -R grader:grader /var/www/ItemCatalog-FSDN
```

# Deploy the Item Catalog Project as a WSGI App
- In your project folder create a **project.wsgi** file

```
$sudo nano project.wsgi
# File Content
import sys

sys.path.insert(0, "/var/www/ItemCatalog-FSDN")

from Project import app as application

application.config['SQLALCHEMY_DATABASE_URI'] = (
  'postgresql://catalog:grader@localhost/catalog')
```
- Edit Project.py and database_setup.py files to connect to Postgre database

```
engine = create_engine(
  'postgresql://catalog:grader@localhost/catalog')
```
- Configuring Apache

```
$cd /etc/apache2/sites-available
$sudo nano ItemCatalog-FSDN.conf


# File content
<VirtualHost *:80>
        ServerName 18.195.218.241

        WSGIDaemonProcess moi user=grader group=grader threads=5

        WSGIScriptAlias / /var/www/ItemCatalog-FSDN/project.wsgi
        <Directory /var/www/ItemCatalog-FSDN/>
                WSGIApplicationGroup %{GLOBAL}
                Order deny,allow
                Allow from all
        </Directory>
</VirtualHost>


$sudo a2ensite ItemCatalog-FSDN.conf
$sudo service apache2 restart
```
- Making sure **.git** directory is not publicly accessible via a browser

```
$cd /etc/apache2/sites-available
$sudo nano ItemCatalog-FSDN.conf

# Add these lines to file content
<Directory /var/www/ItemCatalog-FSDN/.git>
    order allow,deny
    deny from all
</Directory>
```
- Install required packages

```
$sudo pip3 install flask
$sudo pip3 install oauth2client
$sudo pip3 install requests
$sudo pip3 install sqlalchemy
```
- Troubleshooting

```
$sudo tail /var/log/apache2/error.log
```


## Resources
- <a href='https://help.ubuntu.com/community/$sudoers'>Ubuntu Documentation: $sudoers</a>
- <a href='https://websiteforstudents.com/how-to-disable-remote-logon-for-root-on-ubuntu-16-04-lts-servers/'> How To Disable Remote Logon For Root On Ubuntu 16.04 LTS Servers</a>
- <a href='https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt'>Configure the local timezone to UTC</a>
- <a href='https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04'>How To Install and Use PostgreSQL on Ubuntu 16.04</a>
- <a href='http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/'>mod_wsgi (Apache)</a>
- <a href='http://httpd.apache.org/docs/current/mod/mod_alias.html#alias'>Apache Module mod_alias</a>

