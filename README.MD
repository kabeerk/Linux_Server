# Linux Based Server Configuration Project

Host name: http://ec2-52-66-38-19.ap-south-1.compute.amazonaws.com/
Project IP: 52.66.38.19

This file reviews the steps taken to launch a project named Catalog using Amazon Lightsail'sremote Linux  server.

## Step 1:

Create an instance on Amazon Lightsail. Our app was launched in a Linux environment using Ubuntu with Mumbai as the datacenter.

Download the keys that Amazon Lightsail generates for you to your computer and store them in  `/Users/USER_NAME/.ssh`

Note that USER_NAME above will be your own username in your local machine.

## Step 2:

Copy your public IP address from the Lightsail page. This document refers to  `52.66.38.19`.

Using Git Bash, log into your server using the following command:

```
ssh -i ~/.ssh/catalog_root ubuntu@52.66.38.19
```

Note that ~ is the full path to your key file.

## Step 3:

To update currently installed packages, run the following command-

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install unattended-upgrades
sudo unattended-upgrades
```

## Step 4:

Per project requirements, we must change the ssh port to 2200. 

Open your ssh config file to edit it using

```
sudo nano /etc/ssh/sshd_config
```

Locate "Port 22" and replce it with "Port 2200".

Restart the service with

```
sudo service ssh restart
```

## Step 5:

We then have to configure the UFW to allow the connections per the porject instructions. See below:

```
sudo ufw allow 2200
sudo ufw allow 80
sudo ufw allow 123
```

Enable the firewall

```
sudo ufw enable
```


The firewall has now been enable. To test it, exit it and try to re-connect using port 2200

```
ssh -p 2200 -i ~/.ssh/catalog_root ubuntu@52.66.38.19
```

## Step 6:

We then create the user `grader`:


```
sudo adduser grader
```

You will be asked to fill in a few details, simply add the name and use `password` for a password (or anything that you will remember).

The user should now be create.

## Step 7:

The user needs to be given sudo permission, so create the `grader  ` file inside `sudoers.d`.

```
touch /etc/sudoers.d/grader
```

Edut the file and add the following using `nano`:


```
(`sudo nano /etc/sudoers.d/grader`)

```
and add the following:

```
grader ALL=(ALL) NOPASSWD:ALL
```

## Step 8:

Create a key-pair for the user `grader` by running ssh-keygen on your local machine. For this project, the file was named catalog_grader.

Copy the generated key from the `.pub` file and on the terminal accessing the server run:

```
su - grader
mkdir .ssh
chmod 700 .ssh
```

Paste the contents into the generated file

```
nano .ssh/authorized_keys
# paste the contents and save the file
chmod 644 .ssh/authorized_keys
```

You should then restart the service.

```
sudo service ssh restart
```

Exit the connection and login as `grader`:

```
ssh -p 2200 -i ~/.ssh/catalog_grader grader@52.66.38.19
```

## Step 9:

Now configure the timezone using:

```
sudo dpkg-reconfigure tzdata
```

We will configure it to be UTC. First select None of the above and then select UTC.

## Step 10:

We now have to install a few components to run our app using Apache. Run the following commnads to install what you need.

```
sudo apt-get install python
sudo apt-get install python-setuptools
sudo apt-get install apache2 libapache2-mod-wsgi
```

Start the apache service using:

```
sudo service apache2 restart
```

## Step 11:

We will need to install postgresql to manage our database.

```
sudo apt-get install postgresql
```

Activate the psql shell

```
sudo -u postgres psql
```


Run the following to create a user named `catalog`. Enter a password you can remember, 'password' is used here.

```
create user catalog with password 'password';
create database catalog with owner catalog;
```


Exit psql:

```
\q
```

## Step 12:

Install git in order to clone your repo:

```
sudo apt-get install git
```

## Step 13:

This project uses https://github.com/kabeerk/catalog.git

Clone it onto the server

```
cd /var/www
sudo git clone https://github.com/kabeerk/catalog.git
```

Setup the project by installing the necessary modules

```
sudo easy_install pip
cd catalog
export DB_URI=postgresql://catalog:password@localhost/catalog
python database_setup.py
```

## Step 14:

Create a configuration file to run your project using Apache.

```
sudo nano /etc/apache2/sites-available/catalog.conf
```



Include the following content:

```
<VirtualHost *:80>
    ServerName 52.66.38.19

    WSGIScriptAlias / /var/www/catalog/wsgi.py

    <Directory /var/www/catalog>
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```

You wsgi file should include the below content

```
import sys

sys.path.insert(0, '/var/www/catalog')


from catalog import app as application

application.secret_key = 'YOUR_KEY'

application.config['SQLALCHEMY_DATABASE_URI'] = (
    'postgresql://'
    'catalog:password@localhost/catalog')
```

Now enable the site and restart Apache:

```
sudo a2ensite catalog
sudo service apache2 reload
```

The server should be live now. Visit the IP to check (http://52.66.38.19). 

## Step 15:

Set up Google OAuth by adding your origin and redirects to the OAuth server. Note that you will need to save your clients_secrets.json file in your linux server once you make your updates.


## Step 16:

Disable root login & password-based login through ssh in the config file:

```
sudo nano /etc/ssh/sshd_config
```

Change the values to match the two items below:

```
PermitRootLogin no
PasswordAuthentication no
```

Save and restart the server


```
sudo service ssh restart
```