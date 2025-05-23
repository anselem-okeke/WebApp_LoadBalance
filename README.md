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
> # Setup and Provisioning should be done in the order as seen below:  
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





