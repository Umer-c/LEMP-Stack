# LEMP-Stack
LEMP stack project with Horizontal scaling

In this project we will be deploying 5 servers seperately. I have vagrant to create these ubuntu servers. Nginx server will be searving as loadbalancer having ip hash active for persistant sessions for the users. On the other side we have two wordpress servers connected with another backend db server. Here is an overview of the stack.

<img width="962" alt="image" src="https://github.com/Umer-c/LEMP-Stack/assets/73327307/d4fec030-ac7d-44dd-8ea0-23f7a7e1af2f">

## Nginx Server Configuration

Installing NGINX on a Linux based system is fairly straightforward. Following the below commands you can easily set it up.

```
sudo apt update && sudo apt upgrade -y # we update the packages and the operating system
sudo apt install nginx -y # we install nginx and automatically validate with the -y flag
```

Upon completion of the installation, NGINX should automatically register as a service with systemd. So we need to start the service and also ensure that it can run on system restart

```
sudo systemctl enable nginx # start the service on every system reboot
sudo systemctl start nginx # start the service now
sudo systemctl status nginx # check the status of the nginx service
```
So we can go to the server's ip address to validate that everything is working. We need to get to the default page of nginx

![image](https://github.com/Umer-c/LEMP-Stack/assets/73327307/82c6b65d-8686-40ce-b965-d3572fb71b53)

### NGINX Configuration

NGINX configuration files are located in the /etc/nginx/ directory and carry the .conf extension. Essentially, all we have to do is configure NGINX with instructions so that we know what type of connections to listen for and where to redirect them. Let's create a new configuration file using the text editor of our choice.

```
sudo vim /etc/nginx/conf.d/load-balancer.conf
```
The configuration file for me is like this, I have used ip hash so the user's sessions are persistant. The ips mentioned belong to my wordpress servers.
![image](https://github.com/Umer-c/LEMP-Stack/assets/73327307/19227302-d653-406d-8f6a-fa87a21f601f)

To check the syntex and restart the nginx services
```
sudo nginx -t
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```

## Maria DB Setup

MariaDB is a widely used open-source relational database management system, derived from MySQL in 2009. It offers enhanced performance, security, and replication capabilities compared to MySQL. MariaDB has three main editions: Community Server, Enterprise Server, and SkySQL, catering to different levels of scalability, support, and cloud-based solutions.
Let's start the installation

```
sudo apt update -y
sudo apt install mariadb-server mariadb-client -y
sudo apt install -y software-properties-common
sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://mariadb.mirror.liquidtelecom.com/repo/10.6/ubuntu focal main'
sudo apt update && sudo apt install -y mariadb-server mariadb-client
mariadb --version
```

By default, the MariaDB database engine starts automatically during installation. We can verify this by running the command:

```
sudo systemctl status mariadb
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

The MariaDB database comes to us with default settings that are considered present security holes that can potentially be exploited by malicious parties leading to data breaches. To resolve this issue, we need to take an additional step and harden our MariaDB instance. To improve the security of the MariaDB database engine, we need to run the mysql_secure_installation shell script as follows and then follow the on screen instructions as per your need

```
sudo mysql_secure_installation
```
We will now create a new user account in the database server with authentication by password and later assign privileges to the user, this is a good practice in order not to use the root user permanently as it is not secure.

```
sudo mariadb -u root -p
Enter password:
CREATE DATABASE wordpress;
CREATE USER 'wordpress'@'192.168.10.11' IDENTIFIED BY 'wordpress';
CREATE USER 'wordpress'@'192.168.10.12’ IDENTIFIED BY 'wordpress';
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER
    -> ON wordpress.*
    -> TO wordpress@192.168.10.11; #And do the same for 192.168.10.12
FLUSH PRIVILEGES;
EXIT;
```
you will need to comment out the line bind-address = 127.0.0.1 and skip-networking (if it exists)

```
sudo vim /etc/mysql/mariadb.conf.d/50-server.cnf
```

## WordPress Configuration (Server1 & Server2)

WordPress is a free, open source content management system primarily used to publish blogs on the Internet. WordPress simplifies the creation and maintenance of websites and blogs. Due to its popularity, more than a third of all websites today are powered by WordPress. It is written in PHP and uses MariaDB and/or MySQL as its database. Lets configure Wordpress server1 & server2

```
sudo apt-get install php php-fpm php-curl php-mysql php-gd php-mbstring php-xml php-imagick php-zip php-xmlrpc -y
php -v
```
Next, let's modify the PHP configuration file and change some default settings:

```
sudo nano /etc/php/7.4/fpm/php.ini

#Perform the following changes

cgi.fix_pathinfo=0
upload_max_filesize = 128M
post_max_size = 128M
memory_limit = 512M
max_execution_time = 120
```
Let's download the wordpress.

```
cd /var/www/html
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -zxvf latest.tar.gz
sudo mv /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php
```

Next, edit the WordPress configuration file and set the parameters for our database:

```
Sudo vim /var/www/html/wordpress/wp-config.php

#Change the following
define('DB_NAME', 'wordpress' ); # database name

/** Database USER_NAME */ # name of the user
define( 'DB_USER', 'wordpress' );

/** Database password */ # user's password
define( 'DB_PASSWORD', ‘wordpress’ );

/** Database hostname */ # the location of our database which is located on the same server as the website so "localhost
define( 'DB_HOST', ‘192.168.10.13’ );
```
Save and close the file when you are done. Next, set the appropriate permission and ownership for the WordPress directory:

```
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress
Sudo systemctl start apache2
```
Once done we should be able to access the wordpress home/setup page at our specifiec domain

<img width="802" alt="image" src="https://github.com/Umer-c/LEMP-Stack/assets/73327307/99f0a13b-71fd-401e-ae96-c4fd6ae1e323">

## Enable HTTPS on WordPress with Let's Encrypt

Let's Encrypt is an organization that acts to validate the identity of entities such as websites, email addresses, businesses or individuals and bind them to cryptographic keys through the publication of electronic documents called digital certificates. Adopting the HTTPS protocol on the web is easier with the help of its certificates. Since its launch in late, 2015, Let's Encrypt has become the world's largest certificate authority, representing more currently valid certificates than all other browser-approved certificate authorities combined. As of January 2019, it had issued more than 538 million certificates for 223 million domain names.

To enable the HTTPS protocol on our site, we need to install the Certbot client from Let's Encrypt on our system. We can install it by running the following command:

```
sudo apt-get install python3-certbot-nginx -y
```
Once the Certbot client is installed, let's run the following command to enable HTTPS on our site and follow the on screen instructioins.

```
sudo certbot --nginx -d wordpress.uasghar.cloudns.ch
```
we should be able to see the below certificate and now we can access our website/server at https instead of http:

![image](https://github.com/Umer-c/LEMP-Stack/assets/73327307/f461335f-3784-4e3e-afa7-7ec00debd6c1)





