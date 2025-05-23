# NGINX-Based Scalable System Architecture
This project demonstrates a scalable web system architecture leveraging NGINX as a reverse proxy and load balancer, backed by Apache Tomcat for application hosting, RabbitMQ for asynchronous messaging, and MySQL for persistent data storage.

## System Architecture Diagram
![System Architecture](images/system_architecture.png)


## ⚙️ Components Overview

| Component        | Description |
|------------------|-------------|
| **Users**        | External clients interacting with the application. |
| **Load Balancer**| Distributes incoming traffic to NGINX servers for redundancy and performance. |
| **NGINX**        | Acts as a reverse proxy and load balancer, routing requests to the backend application. |
| **Tomcat (Apache)** | Hosts the core web application logic and interfaces `Appication Server`. |
| **RabbitMQ**     | Message broker enabling asynchronous communication between services. |
|**Memcache**| Responsible for Database Caching. |
|**ElasticSearch**| Indexing/Search service. |
| **MySQL**        | Relational database for data persistence. |
| **NFS** (Optional)| Shared file system for web application assets or logs (if applicable). |


---

## Traffic Flow

1. Users send HTTP requests to the Load Balancer.
2. Requests are routed to one of the NGINX servers.
3. NGINX proxies the requests to the Apache Tomcat application servers.
4. Application logic may interact with:
   - RabbitMQ for asynchronous jobs
   - MySQL for data storage
   - NFS (optional) for shared file operations

---

## Tech Stack / Provisioning

- **NGINX**
- **Apache Tomcat**
- **Memcache**
- **RabbitMQ**
- **ElasticSearch**
- **MySQL**
- **NFS (optional)**
- **Linux (1-Ubuntu-based and 4-CentOS-based deployment) `Vagrant file`**

---

## Use Cases

- High availability and load-balanced web platforms
- Message-driven architecture for scalability
- Decoupled frontend/backend services using RabbitMQ
- Secure and efficient reverse proxy setup

---

## Setup Instructions
> ### Setup and Provisioning should be done in the order as seen below:  
>1. **MySQL** – Database service  
>2. **Memcache** - DB Caching
>3. **RabbitMQ** – Messaging broker/Queque 
>4. **Apache Tomcat** – Web application 
>5. **NGINX** – Reverse proxy/load balancer
>6.  **Five VM machine, 1-Ubuntu-based and 4-CentOS-based `vagrant file`**
>7.  **The provisioning of the vm's are in a way that they talk to each other through ssh**

>### 1. MySQL Setup
> **Login to the DB VM**
> 
> ```bash
> vagrant ssh db01
> ```
> **Verify Hosts entry, if entries missing update the it with IP and hostnames**
> 
> ```shell
> cat /etc/hosts
>```
> **Update OS**
> 
> ```shell
> dnf update -y
> ```
> **Set Repository**
> ```shell
> dnf install epel-release -y
> ```
> 
> 
> **Install git and Maria DB Package**
>```shell
> dnf install git mariadb-server -y
>```
> **Starting & enabling mariadb-server**
> ```shell
> systemctl start mariadb
> systemctl enable mariadb
> ```
> **RUN mysql secure installation script, Set db root password, as admin123 for practice purpose**
> 
> ```shell
>  mysql_secure_installation
> ```
> **Secure MySQL Installation Output**
>
> ```plaintext
> Set root password? [Y/n] Y
> New password:
> Re-enter new password:
> Password updated successfully!
> Reloading privilege tables..
> ... Success!
> 
> By default, a MariaDB installation has an anonymous user, allowing anyone
> to log into MariaDB without having to have a user account created for
> them. This is intended only for testing, and to make the installation
> go a bit smoother. You should remove them before moving into a
> production environment.
> 
> Remove anonymous users? [Y/n] Y
> ... Success!
> 
> Normally, root should only be allowed to connect from 'localhost'. This
> ensures that someone cannot guess at the root password from the network.
> 
> Disallow root login remotely? [Y/n] n
> ... skipping.
> 
> By default, MariaDB comes with a database named 'test' that anyone can
> access. This is also intended only for testing, and should be removed
> before moving into a production environment.
> 
> Remove test database and access to it? [Y/n] Y
> - Dropping test database...
> ... Success!
> - Removing privileges on test database...
> ... Success!
> 
> Reloading the privilege tables will ensure that all changes made so far
> will take effect immediately.
> 
> Reload privilege tables now? [Y/n] Y
> ... Success!
> ```
> **Set DB name and users**
> ```shell
> mysql -u root -padmin123
> ```
> ```text
> mysql> create database accounts;
> mysql> grant all privileges on accounts.* TO 'admin'@'localhost' identified by 'admin123';
> mysql> grant all privileges on accounts.* TO 'admin'@'%' identified by 'admin123';
> mysql> FLUSH PRIVILEGES;
> mysql> exit;
>```
> **Download Source code & Initialize Database**
> ```text
> cd /tmp/
> git clone https://github.com/anselem-okeke/WebApp_LoadBalance.git
> cd WebApp_LoadBalance
> mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
> mysql -u root -padmin123 accounts
> mysql> show tables;
> mysql> exit;
>```
> **Restart mariadb-server**
> ```shell
> systemctl restart mariadb
>```
> **Starting the firewall and allowing the mariadb to access from port no. 3306**
> ```text
> systemctl start firewalld
> systemctl enable firewalld
> firewall-cmd --get-active-zones
> firewall-cmd --zone=public --add-port=3306/tcp --permanent
> firewall-cmd --reload
> systemctl restart mariadb
>```

>### 2. MEMCACHE SETUP
> **Login to the Memcache vm**
> ```shell
> vagrant ssh mc01
> ```
> **Verify Hosts entry, if entries missing update the it with IP and hostnames**
> ```shell
> cat /etc/hosts
> ```
> **Update OS with latest patches**
>```shell
>dnf update -y
>```
> **Install, start & enable memcache on port 11211**
> ```text
> sudo dnf install epel-release -y
> sudo dnf install memcached -y
> sudo systemctl start memcached
> sudo systemctl enable memcached
> sudo systemctl status memcachedsed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
> sudo systemctl restart memcached
> ```
> **Starting the firewall and allowing the port 11211 to access memcache**
> ```text
> systemctl start firewalld
> systemctl enable firewalld
> firewall-cmd --add-port=11211/tcp
> firewall-cmd --runtime-to-permanent
> firewall-cmd --add-port=11111/udp
> firewall-cmd --runtime-to-permanent
> sudo memcached -p 11211 -U 11111 -u memcached -d
> ```

>### 3. MEMCACHE SETUP
> **Login to the RabbitMQ vm**
>```shell
> vagrant ssh rmq01
>```
>**Verify Hosts entry, if entries missing update the it with IP and hostnames**
>```shell
>cat /etc/hosts
>```
> **Update OS with latest patches**
>```shell
> dnf update -y
>```
> **Set EPEL Repository**
>```shell
>dnf install epel-release -y
>```
> **Install Dependencies**
> ```text
> sudo dnf install wget -y
> dnf -y install centos-release-rabbitmq-38
> dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
> systemctl enable --now rabbitmq-server
> ```
> **Setup access to user test and make it admin**
> ```text
> sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
> sudo rabbitmqctl add_user test test
> sudo rabbitmqctl set_user_tags test administrator
> rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
> sudo systemctl restart rabbitmq-server
>```
> **Starting the firewall and allowing the port 5672 to access rabbitmq**
> ```shell
> sudo systemctl start firewalld
> sudo systemctl enable firewalld
> firewall-cmd --add-port=5672/tcp
> firewall-cmd --runtime-to-permanent
> sudo systemctl start rabbitmq-server
> sudo systemctl enable rabbitmq-server
> sudo systemctl status rabbitmq-server
>```




> ### 4.TOMCAT SETUP
> **Login to the tomcat vm**
> ```shell
> vagrant ssh app01
>```
> **Verify Hosts entry, if entries missing update the it with IP and hostnames**
> ```shell
> cat /etc/hosts
> ```
> Update OS with latest patches 
> ```shell
> dnf update -y
> ```
> Set Repository 
> ```shell
> dnf install epel-release -y
>```
> Install Dependencies
> ```shell
> dnf -y install java-17-openjdk java-17-openjdk-devel
> dnf install git wget -y
>```
> Change dir to /tmp 
> ```shell
> cd /tmp/
> ```
> Download & Install Tomcat Package
> ```shell
> wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.26/bin/apache-tomcat-10.1.26.tar.gz
> tar xzvf apache-tomcat-10.1.26.tar.gz
>```
> Add tomcat user 
> ```shell
> useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
>```
> Copy data to tomcat home dir 
> ```shell
> cp -r /tmp/apache-tomcat-10.1.26/* /usr/local/tomcat/
>```
> Make tomcat user owner of tomcat home dir 
> ```shell
> chown -R tomcat.tomcat /usr/local/tomcat
>```
> ### Setup systemctl command for tomcat
> Create tomcat service file
> ```shell
> vi /etc/systemd/system/tomcat.service
>```
> Update the file with below content
> ```text
> [Unit]
> Description=Tomcat
> After=network.target
> 
> [Service]
> User=tomcat
> Group=tomcat
> WorkingDirectory=/usr/local/tomcat
> Environment=JAVA_HOME=/usr/lib/jvm/jre
> Environment=CATALINA_PID=/var/tomcat/%i/run/tomcat.pid
> Environment=CATALINA_HOME=/usr/local/tomcat
> Environment=CATALINE_BASE=/usr/local/tomcat
> ExecStart=/usr/local/tomcat/bin/catalina.sh run
> ExecStop=/usr/local/tomcat/bin/shutdown.sh
> RestartSec=10
> Restart=always
> 
> [Install]
> WantedBy=multi-user.target
>```
> Reload systemd files 
> ```shell
> systemctl daemon-reload
>```
> Start & Enable service
> ```shell
> systemctl start tomcat
> systemctl enable tomcat
>```
> Enabling the firewall and allowing port 8080 to access the tomcat
> ```text
> systemctl start firewalld
> systemctl enable firewalld
> firewall-cmd --get-active-zones
> firewall-cmd --zone=public --add-port=8080/tcp --permanent
> firewall-cmd --reload
>```


> ### CODE BUILD & DEPLOY (app01)
> Maven Setup
> ```text
> cd /tmp/
> wget https://archive.apache.org/dist/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.zip
> unzip apache-maven-3.9.9-bin.zip
> cp -r apache-maven-3.9.9/ /usr/local/maven3.9
> export MAVEN_OPTS="-Xmx512m"
>```
> Download Source code
> ```shell
> git clone https://github.com/anselem-okeke/WebApp_LoadBalance.git
>```
> Update configuration
> ```text
> cd WebApp_LoadBalance
> vim src/main/resources/application.properties
> Update file with backend server details
>```
> ### Build code
> ### Run below command inside the repository (WebApp_LoadBalance)
> ```shell
> /usr/local/maven3.9/bin/mvn install
>```
> ### Deploy artifact
> ```text
> systemctl stop tomcat
> rm -rf /usr/local/tomcat/webapps/ROOT*
> cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
> systemctl start tomcat
> chown tomcat.tomcat /usr/local/tomcat/webapps -R
> systemctl restart tomcat
>```

> ### 5.NGINX SETUP
> Login to the Nginx vm
> ```shell
> vagrant ssh web01
> sudo -i
>```
> Verify Hosts entry, if entries missing update the it with IP and hostnames
> ```shell
> cat /etc/hosts
> ```
> Update OS with latest patches
> ```shell
> apt update && apt upgrade -y
>```
> Install nginx 
> ```shell
> apt install nginx -y
> ```
> Create Nginx conf file 
> ```shell
> vi /etc/nginx/sites-available/vproapp
> ```
> Update with below content
> ```nginx configuration
> upstream vproapp {
>   server app01:8080;
> }
> server {
>     listen 80;
>     location / {
>     proxy_pass http://vproapp;
>   }
> }
>```
> Remove default nginx conf
> ```shell
> rm -rf /etc/nginx/sites-enabled/default
> ```
> Create link to activate website
> ```shell
> ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
> ```
> Restart Nginx 
> ```shell
> systemctl restart nginx
> ```




