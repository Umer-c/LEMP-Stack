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


