### PROJECT 8
### LOAD BALANCER SOLUTION WITH APACHE

After the completion of the previous project-7 which was on tooling web solution one may start wondering how a user will have access to each of the webservers 
that has three different IP addresses and three different DNs names. It literally begs the question what the real reasons behind is setting up three servers i
n the first place, one may be inclined to think is more of a luxury that necessity when the three servers are doing
the same job functions.

Clients do access the websites from the browser through the URL but are not aware of what goes behind the scenes of those websites, how many servers 
are serving our requests, these are facts and issues that are not shown to the regular users. But in case of websites that are being visited by millions of users per day
(like google or reddit) it is impossible to serve all the users from a single web server (it is also applicable to databases, but for now i will not focus
on distributed dbs).

However, each URL contains a (DNS) domain name part, which is interpreted (resolved) to IP address of a target server that will serve requests when open 
a website in the Internet. This interpretation (resolution) of domain names is performed by DNS servers, the most used one has a public IP address 8.8.8.8 and
belongs to Google.Let’s look at this scenario the load on one webserver can increase necessitating the need for more servers and more customers, you can add
more CPU and RAM or completely replace the server with a more powerful one – this is called "vertical scaling". This approach has limitations – at some point you reach 
the maximum capacity of CPU and RAM that can be installed into your server.

Another case scenario approach used to cater for increased traffic is "horizontal scaling" – distributing load across multiple Web servers.
This approach is much more common and can be applied almost seamlessly and almost infinitely (one can only imagine how many server
Google has to serve billions of search requests).

Horizontal scaling allows to adapt to current load by adding (scale out) or removing (scale in) Web servers. Adjustment of number of servers can be done manually
or automatically (for example, based on some monitored metrics like CPU
and Memory load).Property of a system (in our case it is Web tier) to be able to handle growing load by adding resources, is called "Scalability".

In our set up in Project-7 we had 3 Web Servers and each of them had its own public IP address and public DNS name. A client has to access them by using different URLs,
which is not a nice user experience to remember addresses/names of even 3 servers, let alone millions of Google servers. To hide all this complexity and to have a single
point of access with a single public IP address/name, a Load Balancer can be used. A Load Balancer (LB) distributes clients’ requests among underlying Web Servers
and makes sure that the load is distributed in an optimal way.
Let us look at the updated solution architecture with an LB added on top of Web Servers (for simplicity let us assume it is a software L7 Application LB,
for example– Apache, NGINX or HAProxy)

![image](https://user-images.githubusercontent.com/55473846/144192081-f3090525-61a0-4cd5-92f6-f2eeb770b318.png)

In this project we will enhance our Tooling Website solution by adding a Load Balancer to distribute traffic between Web Servers and allow users to access our website
using a single URL.

Task
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 instance. Make sure that users can be served by
Web servers through the Load Balancer. To simplify, let us implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers.

Prerequisites

I made sure that i have the following servers installed and configured within Project-7:

1.	Two RHEL8 Web Servers
2.	One MySQL DB Server (based on Ubuntu 20.04)
3.	One RHEL8 NFS server


![image](https://user-images.githubusercontent.com/55473846/144192355-17419c0f-a639-4792-a9ab-296e3fcf3e2a.png)

PREPARING THE SECOND WEBSERVER (WEB2)

Sudo yum -y update

![image](https://user-images.githubusercontent.com/55473846/144192468-261d7f67-1f98-464b-8558-df5f0ae2ffa4.png)

sudo yum install httpd -y

![image](https://user-images.githubusercontent.com/55473846/144192573-93455bd7-f5b6-4fd7-8f97-3b944238da94.png)

which httpd

sudo systemctl restart httpd

sudo systemctl enable httpd

sudo systemctl status httpd

![image](https://user-images.githubusercontent.com/55473846/144192739-83f2641e-688f-4c1e-a841-3b87e1692331.png)

sudo ls -l /var/log

![image](https://user-images.githubusercontent.com/55473846/144192805-22ff2b07-3c15-4437-92f5-307a6b8ff999.png)

sudo yum install git


![image](https://user-images.githubusercontent.com/55473846/144192906-bd270cc0-37d5-4f64-a097-450cfd347b1c.png)

 
git clone https://github.com/darey-io/tooling.git


![image](https://user-images.githubusercontent.com/55473846/144193070-cb768123-080a-4126-bc99-19a46d3e8fc4.png)

CONFIGURE APACHE AS A LOAD BALANCER

I configured Apache as a Load Balancer
I created an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this:
Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.

Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

#Install apache2

sudo apt update

sudo apt install apache2 -y

sudo apt-get install libxml2-dev

#Enable following modules:

sudo a2enmod rewrite

sudo a2enmod proxy

sudo a2enmod proxy_balancer

sudo a2enmod proxy_http

sudo a2enmod headers

sudo a2enmod lbmethod_bytraffic

#Restart apache2 service

sudo systemctl restart apache2

![image](https://user-images.githubusercontent.com/55473846/144193305-51bd4728-c75c-4228-a3ed-fca55e01c139.png)

![image](https://user-images.githubusercontent.com/55473846/144194642-b803f727-ac1a-4d4a-8b0d-bc9401a1051f.png)

I made sure apache2 is up and running


![image](https://user-images.githubusercontent.com/55473846/144194752-34454210-5e92-4312-b846-86d8bb62002f.png)


![image](https://user-images.githubusercontent.com/55473846/144194819-3dc9ba71-f5ea-4b0b-85e2-b4c8318066ef.png)

![image](https://user-images.githubusercontent.com/55473846/144194870-8b1dffdd-3c21-48cd-8a3d-3146bf1ceb87.png)

Make sure apache2 is up and running

sudo systemctl status apache2

Configuring load balancing on the loan balancer server I ran

sudo vi /etc/apache2/sites-available/000-default.conf

and I entered the below configuration inside the <VirtualHost *:80>  </VirtualHost>


#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

#Restart apache server


![image](https://user-images.githubusercontent.com/55473846/144195012-cc1ea3ab-e09e-4581-8650-c743011786f8.png)
    
  
#Restart apache server

sudo systemctl restart apache2

 ![image](https://user-images.githubusercontent.com/55473846/144195110-2c714699-f6d2-4680-ba29-498a7269842d.png)
    
   Troubleshooting I ran the command below with 
    
sudo apachectl configtest
    
by traffic balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion
 the traffic must be distributed by load factor parameter.

  ![image](https://user-images.githubusercontent.com/55473846/144196156-be20e8a8-f233-42ad-9eaa-379b869f110d.png)
    
    In Project-7 i mounted /var/log/httpd/ from the Web Servers to the NFS server –  I first unmount them and made sure that each Web Server has its own log directory when I ran the command below 
/var/log/httpd/access_log

I verified that my configuration works – try to access your LB’s public IP address or Public DNS name from your browser:
    
I tried to refresh my browser load balancer page  http://35.178.182.17/index.php
 
several times and make sure that both webservers receive HTTP GET requests from your LB – new records must appear in each server’s log
 
 file. The number of requests to each server was approximately the same since we set loadfactor to the same value for both servers 
 
 – it means that traffic will be distributed evenly between them.
 
After trying to connect to the two webservers (web3) and webserver (web2) through the load balancer through the Ip address (
 
 http://35.178.182.17/index.php) .
    
 I got the two webserver page login.php and I was able to login to connect to the database server as show below
    
http://35.178.249.181/admin_tooling.php


  ![image](https://user-images.githubusercontent.com/55473846/144196326-7aba004b-b8e3-4b8b-9d9e-1990014cd180.png)
    
    ![image](https://user-images.githubusercontent.com/55473846/144196394-8a6936bc-666f-472d-87f7-79eedaecf9bf.png)
    
    
While the second webserver shown below
    \
http://35.178.182.17/admin_tooling.php
    
    
    
![image](https://user-images.githubusercontent.com/55473846/144196505-67cc1e8b-84e9-43f8-9403-6684daf2f511.png)
    
    I opened two ssh/Gitbash consoles for both Web Servers and run following command:
    
sudo tail -f /var/log/httpd/access_log
    
    
![image](https://user-images.githubusercontent.com/55473846/144196608-1d43fa75-e6aa-496b-a144-cba706a6d3ea.png)
    
    When I ran the below command on the first webserver(web2)
    
sudo tail -f /var/log/httpd/access_log
    
    
![image](https://user-images.githubusercontent.com/55473846/144196719-ccb56617-0e24-4613-9d15-19b99fc9ed51.png)
    
    The above shows that I could access the two web servers through load balancer at different times once I refreshed meaning the load balancer is able to distributes request to the two webservers and users will not even notice that their requests are served by more than one server
Optional Step – Configure Local DNS Names Resolution
Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management.
So what did I do? I configured local domain name resolution. Using
 /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So I configured IP address to domain name mapping for our LB.
#Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
I opened the /etc/host file by using the command below:

sudo vi /etc/hosts added the flowing into it

172.31.30.6  Web1
 
172.31.23.141  Web1
 
I updated the LB config file with those names instead of IP addresses as shown below
 
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
I now tried to curl your Web Servers from LB locally curl http://Web1 or curl http://Web2 
    
Now when I ran curl http://Web1  pls find below as shown the local resolution and the servers cannot be seen by others servers
    
    
![image](https://user-images.githubusercontent.com/55473846/144196867-a1651b07-5fb4-4a4a-ac83-6eece04ea2b7.png)
    
And when I ran curl web2 I found as shown below
    
    
 ![image](https://user-images.githubusercontent.com/55473846/144196962-2d00d1bc-6937-4980-8c77-c444cb7eaa85.png)
    
    However, this is only internal configuration, and it is also local to your LB server, these names will neither be ‘resolvable’ from other servers internally 
    nor from the Internet.
    
Targed Architecture
Now your set up looks like this architecture as shown below
    
![image](https://user-images.githubusercontent.com/55473846/144197110-cc227c2c-27e1-4d03-a16d-0d59becc4f44.png)
    






    




