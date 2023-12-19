# Deploy Spring boot on Ubuntu/Nginx
Instructions for deploying a spring boot application on Ubuntu using nginx as a reverse proxy. Setting up the spring boot project not covered. If you want to set up a quick test project you can go to
[Spring initializr](https://start.spring.io)

## Install Nginx
`sudo apt-get update`

`sudo apt upgrade`

`sudo apt-get install nginx`

`sudo apt-get install systemd`  

Check Nginx version to see if installation was successfull: `nginx -v`  

## Install Java JRE
`sudo apt-get update`

Run: `java -version` to see how to Install Java JRE. 

Check Java JRE version to see if installation was successfull: `java -version`

## Install Java JDK
`sudo apt-get update` 

Run: `javac -version` to see how to Install Java JDK.  

Check Java JDK version to see if installation was successfull: `javac -version`

## Install Maven
`sudo apt-get update`

`sudo apt-get -y install maven`

Check Maven version to see if installation was successfull: `mvn -version`

## Instal PostgreSQL
`sudo apt-get update`

`apt install postgresql postgresql-contrib`

**Step 1**: To enter psql cli to interact with postgres database engine.

`sudo -u postgres psql`

**Step 2**: Setup password access to postgres user.

`\password <username>`
example: `\password postgres`

**Step 3**: Edit postgresql.conf file & change listen address.

When postgresql is been installed, psql allows listening to localhost connections only, & thus block all the connection request from remote TCP/IP connection.<br >

`sudo vim /etc/postgresql/14/main/postgresql.conf`

use down arrow to scroll, search for listen_addresses = “localhost”, which will commented, uncomment it and replace “localhost” with “*” as shown below:<br >

`listen_addresses = '*'`

**Step 4:** Edit postgres pg_hba.conf for remote host access for all IP.

By default, psql connection is only accepted for localhost & it refuses all the request from TCP/IP remote connection. Therefore, we need to allow a user to login from any IP address.

Open pg_hba.conf file & add this 2 lines at bottom of the file.

`sudo vim /etc/postgresql/14/main/pg_hba.conf`

```
#IPv4 Addresses
host all all 0.0.0.0/0 md5
#IPv6 Addresses
host all all ::0/0 md5
```

**Step 5:** Restart postgreSQL

Now, To apply changed made in postgresql.cong & pg_hba.conf, we need to restart postgreSQL service.

`sudo systemctl restart postgresql`

**Step 6**: Check postgresql listening post number

`ss -nlt | grep 5432`

This should give response as running on global 5432, i.e. 0.0.0.0:5432. This means that psql is now enabled for remote access.

**Step 7**: pgadmin 4 connect to remote connection

## Create jar in Spring Boot project
`mvn clean install -DskipTests`

In `/var/www/` enter jar

## Create a service to start the application

In `/etc/systemd/system/` create a `<myservice>.service` file and add the following.  

```

[Unit]
Description=Final Project Backend Java with MOOC theme
After=syslog.target

[Service]
User=dikisaputra15
ExecStart=/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java -jar /var/www/DeployMOOC/mooc.jar
SuccessExitStatus=143
TimeoutStopSec=10
Restart=on-failure
RestartSec=5

[Install] 
WantedBy=multi-user.target

```

Enable systemd service: `sudo systemctl enable <myservice>.service`

Update systemd service: `sudo systemctl daemon-reload`

`sudo systemctl start <myservice>`

`sudo systemctl stop <myservice>`

`sudo systemctl restart <myservice>`

`sudo systemctl status <myservice>`

## Create an Nginx Config file
In `/etc/nginx/sites-available/` create `<my-appname>` file and add the below setup:  

```

server {
        listen 80;
        server_name <ip>/<namaDomain>;

        location / {

                proxy_pass http://localhost:8083;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                rewrite ^/([^/]+)/?$ /$1 break;

                }
}

```
The default port for the Spring boot application is port 8080, if you change it, make sure to update the port in the above configuration.<br >
For example, the configuration above has updated the port to 8083.  

After creating the file run the following command `sudo ln -s /etc/nginx/sites-available/<my-appname> /etc/nginx/sites-enabled` to symlink the file to the sites enabled directory.  

Enable nginx: `sudo systemctl enable nginx`

Now run `sudo systemctl restart nginx` to restart nginx for the above to take affect.

`sudo systemctl start nginx`

`sudo systemctl stop nginx`

`sudo systemctl restart nginx`

`sudo systemctl status nginx`

## Deploy
Run `<myservice>.service` to start the jar: `sudo systemctl start <myservice>`

Run nginx: `sudo systemctl start nginx`

Now just go to one of the paths set in spring boot and it will be up and running.

## SSL/HTTPS

To run the application over HTTPS we will be installing certbot from letsencrypt.  

`sudo apt-get update`  

` sudo apt install certbot python3-certbot-nginx`

Make sure the server name in Nginx uses the domain name.

Automates the process of granting SSL/TLS certificates from Let's Encrypt for websites hosted on Nginx servers:

`sudo certbot --nginx -d "<nama domain>"`

Follow the instructions and complete the certificate installation.  

### FYI

In all the above wherevere something is in `<>` it means add your own value for this param.

Delete Folder/File: `sudo rm -r <nama folder/file>`

Check java path: `update-alternatives --list java`

Displays running Localhost: `dig localhost:5432`

Displays a detailed list of files and directories: `ls -l`

Displays Spring Log: `sudo journalctl -u mooc.service --since "$(date +'%Y-%m-%d 06:00:00')"`

Check status certbot: `sudo systemctl status certbot.timer`

Update SSL Certificate: `sudo certbot renew`

Test Certificate Renewal: `sudo certbot renew --dry-run`

Check Log certbot: `sudo nano /var/log/letsencrypt/letsencrypt.log`

Check psql version: `SELECT version();`

Displays list role: `\du`

Displays list DB: `\l`

Select DB: `\c <db name>`

Displays list table: `\dt`

Delete symlink: `sudo rm <symlink location>`

Example of database connection to psql ubuntu server
```
spring.datasource.url=jdbc:postgresql://localhost:5432/<database name>
spring.datasource.username=postgres
spring.datasource.password=root

```

Header handling when forwarding requests from the frontend for example, through a proxy or load balancer such as **X-Forwarded-For**, **X-Forwarded-Proto**, to a Spring Boot application.
This command was added to application.properties to handle CORS issues.
```
server.forward-headers-strategy=framework
```

