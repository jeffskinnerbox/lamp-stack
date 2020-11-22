!--
Maintainer:   jeffskinnerbox@yahoo.com / www.jeffskinnerbox.me
Version:      0.0.1
-->


<div align="center">
<img src="http://www.foxbyrd.com/wp-content/uploads/2018/02/file-4.jpg" title="These materials require additional work and are not ready for general use." align="center">
</div>


----

# LAMP Stack Vagrant Box
This repository is used to create a Vagrant box supporting an Ubuntu based LAMP Stack.

A LAMP Stack is a group of open-source software that is typically installed together
in order to enable a server to host dynamic websites and web apps written in PHP.
This term is an acronym which represents the **L**inux operating system, with the **A**pache web server.
The site data is stored in a **M**ySQL database, and dynamic content is processed by **P**HP.
MySQL implements the relational model and uses Structured Query Language (better known as SQL) to manage its data.

There is an often used alternative database to MySQL called MariaDB.
MariaDB is a drop-in replacement for MySQL.
It is developed by former members of MySQL team who are concerned that
Oracle might turn MySQL into a closed-source product.

The Vagrant VM uses the default port 3306 for the [MySQL protocol][04].
This port enable remote connections to MySQL.
This allows you to directly connect to MySQL on the Vagrant VM
from an application or MySQL client running on a different system.

## Sources
Sources used to create these installation scripts:

* [How to Install Apache, PHP 7, and MySQL on Ubuntu with Vagrant](https://www.taniarascia.com/how-to-install-apache-php-7-1-and-mysql-on-ubuntu-with-vagrant/)
* [Mysql server on vagrant, virtualbox](https://www.yourtechy.com/technology/mysql-server-vagrant-virtualbox/)
* [How To Install MySQL on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04)
* [How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04)
* [How to Install LAMP Stack on Ubuntu 20.04 Server/Desktop](https://www.linuxbabe.com/ubuntu/install-lamp-stack-ubuntu-20-04-server-desktop)
* [How to Install phpMyAdmin with Apache (LAMP) on Ubuntu 20.04](https://www.linuxbabe.com/ubuntu/install-phpmyadmin-apache-lamp-ubuntu-20-04)
* [How To Install and Secure phpMyAdmin on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-20-04)

* [Automate mysql_secure_installation Using Bash](https://www.vic-l.com/automate-mysql-secure-installation-using-bash/)
* [Automating mysql_secure_installation](https://bertvv.github.io/notes-to-self/2015/11/16/automating-mysql_secure_installation/)

MySQL Tutorial:

* [What is MySQL? A Beginner-Friendly Explanation](https://kinsta.com/knowledgebase/what-is-mysql/)
* [A Basic MySQL Tutorial](https://www.digitalocean.com/community/tutorials/a-basic-mysql-tutorial)



--------



# Step 1: Installing Apache - DONE
The Apache web server is among the most popular web servers in the world.
It’s well documented, has an active community of users,
and has been in wide use for much of the history of the web,
which makes it a great default choice for hosting a website.

Install Apache using Ubuntu’s package manager

```bash
# update and upgrade the package manager
sudo apt-get update && sudo apt-get -y upgrade

# install the latest version of apache
sudo apt-get -y install apache2 apache2-utils
```

Now we want tocheck on the integrity of Apache.
Check to see if Apache is running (via `systemctl status apache2`)
and if it’s not running, use `systemctl` to start it (via `sudo systemctl start apache2`).

```bash
# check status of apache
systemctl status apache2

# do a configuration test on apache
sudo apache2ctl configtest

# check the version number of apache
apache2 -v

# enable apache2 to automatically start at boot time
sudo systemctl enable apache2
```

# Step 2: Updating the Firewall - DONE
You’ll need to adjust your firewall settings to allow HTTP traffic.
`ufw` has different application profiles that you can leverage to set you Apache settings.
Lets list all currently available `ufw` application profiles:

```bash
# list all currently available ufw application profiles
$ sudo ufw app list
Available applications:
  Apache
  Apache Full
  Apache Secure
  CUPS
  OpenSSH
```

Here’s what each of these profiles mean:

* **Apache:** This profile opens only port 80 (normal, unencrypted web traffic).
* **Apache Full:** This profile opens both port 80 (normal, unencrypted web traffic)
and port 443 (TLS/SSL encrypted traffic).
* **Apache Secure:** This profile opens only port 443 (TLS/SSL encrypted traffic).

For now, it’s best to allow only connections on port 80,
since this is a fresh Apache installation and you still don’t have a TLS/SSL certificate
configured to allow for HTTPS traffic on your server.

To only allow traffic on port 80, use the Apache profile:

```bash
# enable ufw without prompt
sudo ufw --force enable

# only allow traffic on port 80 and 22
sudo ufw allow in "Apache"
sudo ufw allow in "OPENSSH"

# verify the chang
$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
Apache                     ALLOW       Anywhere
OpenSSH                    ALLOW       Anywhere
Apache (v6)                ALLOW       Anywhere (v6)
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

You can do a spot check right away to verify that everything.
If you are using the `Vagrantfile` provided here or host computer,
put the following in your browser:

```bash
# put in your host server browser to see apache web page
http://localhost:8306
```

You should get the default Apache2 home page.



--------



# Step 1: Install PHP - DONE
Enter the following command to install PHP and some common PHP modules:

```bash
# install php and some common PHP modules
sudo apt-get -y install php libapache2-mod-php php-mysql php-common php-cli php-common php-json php-opcache php-readline

# check the php version
$ php --version
PHP 7.4.3 (cli) (built: Oct  6 2020 15:47:56) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.3, Copyright (c), by Zend Technologies

# get the php version number
PHP_VERSION=$(php --version | head -n1 | awk '{ print $2 }' | awk -F . '{ print $1 "." $2 }')

# enable the apache php module (using version number from above)
$ sudo a2enmod php$PHP_VERSION
Considering dependency mpm_prefork for php7.4:
Considering conflict mpm_event for mpm_prefork:
Considering conflict mpm_worker for mpm_prefork:
Module mpm_prefork already enabled
Considering conflict php5 for php7.4:
Module php7.4 already enabled

# restart apache web server
sudo systemctl restart apache2
```

# Step X: Test PHP - DONE
To test PHP scripts with Apache server, we'll create a
`info.php` file in the document root directory.

```bash
# create info.php file in the document root directory
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php > /dev/null

# put in your host server browser to see php web page
http://localhost:8306/info.php
```

# Step X: Run PHP-FPM with Apache - DONE
There are basically two ways to run PHP code with Apache web server:
(1) Apache PHP module and (2) PHP-FPM.
In the above steps, the Apache PHP module is used to handle PHP code, which is usually fine.
But in some cases, you need to run PHP code with PHP-FPM instead.oLet's enable that:

```bash
# disable the Apache PHP7.4 module
sudo a2dismod php7.4

# install php-fpm
sudo apt install php7.4-fpm

# enable proxy_fcgi and setenvif module
sudo a2enmod proxy_fcgi setenvif

# enable the /etc/apache2/conf-available/php7.4-fpm.conf file
sudo a2enconf php7.4-fpm

# restart apache for the changes to take effect
sudo systemctl restart apache2
```

Now if you refresh the `info.php` page in your browser,
you will find that "Server API" is changed from "Apache 2.0 Handler" to "FPM/FastCGI",
which means the Apache web server will pass PHP requests to PHP-FPM.

```bash
# for security, delete info.php file now to prevent prying eyes
sudo rm /var/www/html/info.php
```




--------




# Step X: Install MySQL - DONE
Now that you have a web server up and running,
you need to install the database system to be able to store and manage data for your site.
MySQL is a popular database management system used within PHP environments.

I want to install MySQL non-interactively (no prompts), which is going to be required for the Vagrantfile.
To do this, I will use `debconf-set-selections` and `apt-get` will use `debconf-get-selections`
to configure MySQL setting that might normally be prompted for during installation.

Sources of inspiration:

* [Installing MySQL (with Debconf)](https://serversforhackers.com/c/installing-mysql-with-debconf)
* [Pre-loading debconf values for easy installation](http://blog.delgurth.com/2009/01/19/pre-loading-debconf-values-for-easy-installation/)
* [A super-simple Vagrant LAMP stack bootstrap (installable with one command)](https://www.dev-metal.com/super-simple-vagrant-lamp-stack-bootstrap-installable-one-command/)

>**NOTE:** Personally, I don't want to password protest my MySQL database.
>I just want to log into MySQL via the command `sudo mysql`.
>If so, set `DBPASSWD=''` below but be careful since this
>leaves your MySQL installation unsecured.
>Check out the article ["Removing the MySQL root password][03]
>for more ways to setup passwordless entry.

```bash
# set env variables
# use single quotes instead of double quotes to make it work with special-characters
DBHOST='localhost'
DBNAME='mysql-db-name'
DBUSER='mysql-user'
DBPASSWD='mysql-password'

# when installing mysql, this will give password to installer
sudo debconf-set-selections <<< "mysql-server mysql-server/root_password password $DBPASSWD"
sudo debconf-set-selections <<< "mysql-server mysql-server/root_password_again password $DBPASSWD"
```

Now we'll install the MySQL server and client software.
With the work done above, I will not be prompted for user input,
instead it will be take from the `debconf` database.

```bash
# acquire and install mysql and other required software
#sudo apt-get -y install mariadb-server mariadb-client
sudo apt-get -y install mysql-server libmysqlclient-dev build-essential npm
```

After this, MySQL server should be automatically started
(if it’s not running, start it with `sudo systemctl start mysql`)
but you should check its status:

```bash
# check status of mysql
#systemctl status mariadb
systemctl status mysql

#check mysql server version information
#mariadb --version
mysql --version

# enable mmysql to automatically start at boot time
#sudo systemctl enable mariadb
sudo systemctl enable mysql
```

# Step X: Setup Your MySQL Login - DONE
By default, the MySQL package on Ubuntu uses `unix_socket` to authenticate user login,
which basically means you can use username and password of the OS to log into MySQL console.

If you’re mainly using MySQL from the command line,
you can keep the root account protected by a password,
while still avoiding the inconvenience of having to provide the password on the command line.
Just create a `~/.my.cnf` file:

```bash
# setup mysql configure file to avoid inputing password
DBPASSWD='mysql-password'
echo -e "[client]\nuser = root\npassword = $DBPASSWD" >> ~/.my.cnf
```

Now test the login:

```bash
# login to mtsql console
#sudo mariadb
mysql
```

# Step X: MySQL Post-Installation Security (Optional) - DONE NOT
So as you see above, MySQL doesn't require a root password to get console access.
For production use, it is recommended that you run a security script that comes pre-installed with MySQL.
This script will remove some insecure default settings
and lock down access to your database system.
Start the interactive script by running: `sudo mysql_secure_installation`.

You may want to do this via the commandline, and if so,
[this article][02] and [this article][01] could provide what is needed.

# Step X: Install phpMyAdmin
* [How To Install and Secure phpMyAdmin on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-20-04)
* [How To Install and Secure phpMyAdmin on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-18-04)

`phpMyAdmin` is a free and open-source web-based database management tool written in PHP.
`phpMyAdmin` was created so that users can interact with MySQL (or MariaDB) through a web interface.

Description .....
* [A super-simple Vagrant LAMP stack bootstrap (installable with one command)](https://www.dev-metal.com/super-simple-vagrant-lamp-stack-bootstrap-installable-one-command/)

```bash
# set env variables
# use single quotes instead of double quotes to make it work with special-characters
DBPASSWD='mysql-password'
PHPPASSWD='php-password'

# when installing phpmyadmin, this will give password and other data to installer
sudo debconf-set-selections <<< "phpmyadmin phpmyadmin/dbconfig-install boolean true"
sudo debconf-set-selections <<< "phpmyadmin phpmyadmin/app-password-confirm password $PHPPASSWD"
sudo debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/admin-pass password $DBPASSWD"
sudo debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/app-pass password $DBPASSWD"
sudo debconf-set-selections <<< "phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2"
```

Description .....

```bash
# install phpmyadmin
sudo apt-get -y install phpmyadmin php-mbstring php-zip php-gd php-json php-curl
```

Without the ....
you will be asked a few questions in order to configure your installation correctly:

* For the server selection, choose apache2
* Select Yes when asked whether to use dbconfig-common to set up the database
* You will then be asked to choose and confirm a MySQL application password for phpMyAdmin

This installation process adds the phpMyAdmin Apache configuration file into the
`/etc/apache2/conf-enabled/` directory, where it is read automatically.
The last step is to explicitly enable the mbstring PHP extension and restart Apache:

```bash
# enable the mbstring php extension
sudo phpenmod mbstring

# restart apache
sudo systemctl restart apache2
```

Verify PhpMyAdmin installation by going by accessing in a browser:

```bash
# put in your host server browser to see phpmyadmin web page
http://localhost:8306/phpmyadmin
```

The default username is `phpmyadmin`
and the password is the one that you have set in the previous step (i.e. `DBPASSWD`).

phpMyAdmin is now installed and configured.
However, before you can log in and begin interacting with your MySQL databases,
you will need to ensure that your MySQL users have the privileges required for interacting with the program.

# Step X: Adjusting User Authentication and Privileges - DONE NOT
A default username & password are set at the time of installing phpMyAdmin for doing database tasks
(The default username is ‘phpmyadmin’ and the password is the one that you have set in the previous step).
But it is better to use phpMyAdmin by making a connection as root user to perform all type of database operations.

If no password is set for root user when MySQL (or MariaDB) server is installed,
then it is necessary to set a password for the database server later.

* [Install phpMyAdmin on Ubuntu 18.04](https://linuxhint.com/install_phpmyadmin_ubuntu_1804/)

# Step X:



--------



# Build a LAMP Stack Vagrant Box

## Step 1: Build You Box Using Vagrant
First step is to create your box destine to become your new base box via Vagrant.
The Vagrantfile used to create it is in this repository.

```bash
# build the box
vagrant up
```

We now SSH into the box and do any customizing required of it.
We're also going to clean up disk space on the VM
so when we package it into a new Vagrant box, it's as clean as possible.

```bash
# do any required customization

# remove APT cache
sudo apt-get clean

# "zero out" the drive (this is for Ubuntu)
sudo dd if=/dev/zero of=/EMPTY bs=1M
sudo rm -f /EMPTY

# clear the bash history
cat /dev/null > ~/.bash_history && history -c

# shutdown the vm
sudo shutdown -h now
```

## Step X: Repackage the VM into a New Vagrant Base Box
We are going to repackage the server we just created into a new Vagrant Base Box.

```bash
# create the new box
vagrant package --output lamp-stack.box

# install the vagrant box in your local repository
vagrant box add --name lamp-stack ./lamp-stack.box

# check to see the box is in the local repository
vagrant box list

# remove the built box now that its in your local repository
rm lamp-stack.box

# your vm is nolong needed
vagrant destroy
```

Now you have the box and you can use it like any other box
by referencing it in a `Vagrantfile` for a new build.

If you wise to remove the box from the local repository,
use the command `vagrant box remove lamp-stack`.

## Step X: Test the Build
Now lets test if the newly created Vagrant box in fact works.
You can login into the VM using “vagrant” as user name and “vagrant” as a password,
but first we need to initialize our test environment:

```bash
# create your test environment
mkcd ~/tmp/test-lamp-stack

# initialize the vagrant environment
vagrant init lamp-stack

# bring up the vm
vagrant up
```




[01]:https://www.vic-l.com/automate-mysql-secure-installation-using-bash/
[02]:https://www.linuxbabe.com/ubuntu/install-lamp-stack-ubuntu-20-04-server-desktop
[03]:https://medium.com/@benmorel/remove-the-mysql-root-password-ba3fcbe29870
[04]:https://mysqlserverteam.com/mysql-guide-to-ports/
[05]:
[06]:
[07]:
[08]:
[09]:
[10]:
