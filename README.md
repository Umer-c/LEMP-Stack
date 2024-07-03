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

Lets configure Wordpress server1 & server2

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



