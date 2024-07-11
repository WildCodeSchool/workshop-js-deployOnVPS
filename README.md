# Deploying a fullstack js app on a VPS with Nginx 

[⬅ Version Française](./README-FR)


## Introduction

![webhosting](./assets/webhosting.png)

A server is a computer that communicates with other computers to serve them with the information requested by these computers. These computers, also called clients, connect to a server through either a local area network (LAN) or a wide area network (WAN). A server sends and collects information across a network within multiple locations.

A web server is a server on the internet that uses the Hypertext Transfer Protocol (HTTP) to receive requests from a client, such as a browser. It then returns an HTTP response, which can be an HTML webpage or data in JSON format, as used in API calls.

Web servers, essential for data exchange, use HTTP for client-server communication. They consist of both hardware and software, crucial in web development. The software interprets URLs and manages user access to hosted files.

### Deployment overview

![deployement](./assets/deployOverview.png)

The deployment diagram above allows you to visualize the architecture of our server which will host both our react application as well as our express API and our mySQL database.

Let's take a look at the elements that make it up:

- The client, in other words the user of the site on a physical machine
- The server (hardware) in other words, a physical machine connected to the internet, in our case it is a virtual private server (VPS).
- Nginx, a web server (software) which allows us to manage http requests, this one is directly installed on the physical server and is used to redirect to the corresponding service in our case we have two options either our client application (react) or our api (express/harmonia).
- Static files (react)
- A NodeJS server (express/harmonia)
- A database server (mySQL)

### Web server & reverse proxy

![reverseproxy](./assets/reverseproxy.png)

A traditional forward proxy server allows multiple clients to route traffic to an external network. For instance, a business may have a proxy that routes and filters employee traffic to the public Internet. A reverse proxy, on the other hand, routes traffic on behalf of multiple servers.

A reverse proxy effectively serves as a gateway between clients, users, and application servers. It handles all the access policy management and traffic routing, and it protects the identity of the server that actually processes the request.

### Nginx explained

What is Nginx?

According to its documentation, Nginx (pronounced “engine X”) is an HTTP and reverse proxy server, a mail proxy server, and a generic TCP/UDP proxy server, originally written by Igor Sysoev.

Nginx is used for a variety of tasks that contribute to improving Node performance:

- Reverse proxy server: As traffic to your app increases, the best approach to improve performance is to use Nginx as a reverse proxy server in front of the Node.js server to load balance traffic across the servers. This is the core use case of Nginx in Node.js applications

- Stateless load balancing: This improves performance while reducing load on backend services by sending off client requests to be fulfilled by any server with access to the requested file
    
- Cache static contents: Serving static content in a Node.js application and using Nginx as a reverse proxy server doubles the application performance to a maximum of 1,600 requests per second

- Implement SSL/TLS and HTTP/2: Given the recent shift from using SSL/TLS to secure user interactions in Node.js applications, Nginx also supports HTTP/2 connections
    
- Performance tracking: You can keep real-time tabs on the overall performance of your Node.js application using the stats provided on Nginx’s live dashboards
    
- Scalability: Depending on what assets you’re serving, you can take advantage of the full‑featured HTTP, TCP, and UDP load balancing in Nginx to scale up your Node.js application

Nginx currently supports seven scripting languages: Go, Node.js, Perl, PHP, Python, Ruby, and Java Servlet Containers (the last is an experimental module). It enables you to run applications written in different languages on the same server.

## Preparation

### Choosing a vps

In today's digital age, Virtual Private Servers (VPS) have become an integral part of web hosting, development, and server administration. VPS hosting provides the flexibility of a dedicated server while being cost-effective and easy to manage.

As part of this workshop we will order a vps on OVH but there are plenty of equally viable service providers.

To do this, go to this [link](https://www.ovhcloud.com/fr/vps/) and configure your vps as indicated in the screenshot below

![vpsOrder](./assets/ovhVPSOrder.png)

### Choosing a domain name

A domain name (often simply called a domain) is an easy-to-remember name that's associated with a physical IP address on the Internet. It's the unique name that appears after the @ sign in email addresses, and after www. in web addresses.

For order a domain name on OVH you can follow this [link](https://www.ovhcloud.com/fr/domains/)

### Link the domain name to the vps

Once your VPS is ordered you will receive an email with the connection information to it including the IP, from your OVH dashboard you will be able to link your VPS via its IP address to your domain name by configuring what we call a DNS zone.

You can follow the steps below to set up the link between your domain name and your VPS

- Access to your domain from your dashboard :
![ovh dashboard](./assets/ovhDomain.png)

- Access to the DNS zone of your selected domain :
![ovh dns](./assets/ovhDomainDNS.png)

- link your vps ip to your domain
  - Add a new entry to your dns zone :
  ![dns1](dns1.png)
  - Select the `A` dns type entry :
  ![dns2](dns2.png)
  - add your VPS ip to the `cible` input and confirm :
  ![dns3](dns3.png)

Then repeat the same operation, this time specifying in the `subdomain` field the string `api`

This will allow us to host both our react application and our API on the same server, distinguishing them with a distinct domain ans subdomain name.

## Configure your VPS

Connecting a computer to a Virtual Private Server (VPS) is an essential skill for anyone working with web hosting, development, or server administration.

### How to Connect to a VPS

I'll walk you through the steps to connect to a VPS using SSH (Secure Shell)

Secure Shell (SSH) is a cryptographic network protocol used to securely access and manage remote computers and servers over an unsecured network.

SSH provides a secure channel for data communication and authentication, protecting sensitive information from potential eavesdropping, tampering, or unauthorized access.

  1. Open your terminal
  2. On the command line, enter the command : 
```sh 
  ssh your_user@ip_of_your_vps 
```
  3. When prompted, enter your VPS `root` password.

![ssh](./assets/ssh.png)

You’ll know that your connection was successful if you see SSH: your_ip_address_or_hostname in your terminal

You will need to log in to your VPS via SSH, using the IP address, user name and password provided by email when you received your order. 
{:.alert-info}

### Update packages

Next let's clean and update the server

Run the following command :

```sh
apt clean all && sudo apt update && sudo apt dist-upgrade
```

### Install NodeJS

One way to install Node.js that is particularly flexible is to use nvm, the Node version manager. This software allows you to simultaneously install and maintain multiple independent versions of Node.js as well as their associated Node packages.

To install the NVM on your Ubuntu machine, visit the project's GitHub page. Copy the curl command from the README file that appears on the main page. This will give you the most recent version of the installation script:
```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

Running the command above downloads a script and executes it.The script clones the nvm repository to `~/.nvm` and attempts to add the source lines from the snippet below to the correct profile file ( `~/.bash_profile`, `~/.zshrc`, `~ /.profile`or `~/.bashrc`).

We will therefore create a `.bashrc` file so that nvm can work correctly:
```sh
source ~/.bashrc
```

Now you can ask NVM which Node versions are available:
```sh
nvm list-remote
```
Now install and use the version that corresponds to your development environment or the latest stable version of nodejs with the following command:
```sh
nvm install lts
nvm use lts
```

### Install a process manager

A process manager is a tool, which provides an ability to control application lifecycle, monitor the running services and facilitate common system admin tasks to maintain your project operability.

In your terminal connected to your vps enter the following command to install pm2 : 
```sh
npm install pm2 -g
```

PM2 is a daemon process manager that will help you manage and keep your application online 24/7
{:.alert-info}

## Install and configure a database

### Default installation and configuration

You can install MySQL using the APT package repository. 

To install it, update the package index on your server if you haven't done so recently : 

```bash
sudo apt update
```

Next, install the `mysql-server` package :

```bash
sudo apt install mysql-server
```

This will install MySQL, but will not ask you to set a password or make other configuration changes. As this makes your MySQL installation insecure, we will address this point in the next step.


### Authentication configuration

For new installations of MySQL you will need to run the security script included in the DBMS. This script changes some of the less secure default options for things like remote root logins and sample users.

Run the security script with `sudo` :

```bash
sudo mysql_secure_installation
```

You will then be guided through a series of prompts where you can make some changes to the security options of your MySQL installation.

Remember to launch the MySQL service, then check its status :
```bash
sudo service mysql start 
```

![capture_20220312182123974.bmp](./assets/mysqlStart.bmp)

In Ubuntu systems running MySQL 5.7 (and later), the MySQL user **root** is configured to authenticate using the `auth_socket` plugin by default rather than with a password. This helps improve security and usability in many cases, but it can also complicate things if you need to allow an external program (e.g. phpMyAdmin) to access the user.

In order to use a password to connect to MySQL as **root**, you will need to change its authentication method from `auth_socket` to another plugin, such as `caching_sha2_password` or `mysql_native_password`. To do this, open the MySQL prompt from your terminal:

```bash
sudo mysql
```

To configure the **root** account so that it authenticates with a password, from the MySQL prompt issue an `ALTER USER` statement to modify the authentication plugin used and set a new password.

Be sure to replace `password` with a strong password of your choice, and remember that this command will change the **root** password you set in step before:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'password';
```

<aside>
💡 The previous `ALTER USER` statement sets the MySQL user **root** to authenticate with the `caching_sha2_password` plugin. [According to the official MySQL documentation](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html#upgrade-caching-sha2-password), `caching_sha2_password` is MySQL's preferred authentication plugin, as it provides more secure password encryption than the old, but still widely used, `mysql_native_password`

</aside>

<aside>
⚠️ However, many applications do not work reliably with `caching_sha2_password`. If you plan to use this database with a PHP application or connect via an orm like sequelize, you can set **root** to authenticate with `mysql_native_password` instead:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

</aside>

Then run `FLUSH PRIVILEGES` which puts your new changes into effect:

```sql
FLUSH PRIVILEGES;
```

### Create a user and manage permissions

Upon installation, MySQL creates a root user account which you can use to manage your database. This user has full privileges over the MySQL server, meaning it has complete control over every database, table, user, and so on. Because of this, it’s best to avoid using this account outside of administrative functions. This step outlines how to use the root MySQL user to create a new user account and grant it privileges.

Once you have access to the MySQL prompt, you can create a new user with a CREATE USER statement. These follow this general syntax:

```sql
CREATE USER 'username'@'host' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

After CREATE USER, you specify a username. This is immediately followed by an @ sign and then the hostname from which this user will connect. If you only plan to access this user locally from your Ubuntu server, you can specify localhost. Wrapping both the username and host in single quotes isn’t always necessary, but doing so can help to prevent errors.

The general syntax for granting user privileges is as follows:

```sql
GRANT PRIVILEGE ON database.table TO 'username'@'host';
```

The PRIVILEGE value in this example syntax defines what actions the user is allowed to perform on the specified database and table. You can grant multiple privileges to the same user in one command by separating each with a comma. You can also grant a user privileges globally by entering asterisks (*) in place of the database and table names. In SQL, asterisks are special characters used to represent “all” databases or tables.

Run this GRANT statement, replacing `username` with your own MySQL user’s name, to grant these privileges to your user:

```sql
GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'username'@'localhost' WITH GRANT OPTION;
```

Note that this statement also includes WITH GRANT OPTION. This will allow your MySQL user to grant any permissions that it has to other users on the system.

Then run `FLUSH PRIVILEGES` which puts your new changes into effect:

```sql
FLUSH PRIVILEGES;
```

## Install and configure a web server & reverse-proxy

### Default installation and configuration

### Creating a configuration file

### Set up your SSL certificates to manage https access

### Setting up a cron job for our ssl certificates


## Configuration & deployment of a fullstack js app

### Adding minimal security headers

### Set environment variables in production and install dependencies

### Transfer the application to your vps

### Launch your application with pm2

