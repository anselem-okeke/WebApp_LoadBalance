# WebApp_LoadBalancer
## System Architecture
![System Architecture](images/system_architecture.png)

```bash
# Sample: Start RabbitMQ in Docker
docker run -d --hostname rabbit --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management

# Sample: Start MySQL
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 mysql:5.7

```sh
pwd
