# Containerizing Multi-tier Java Web Application
This project demonstrates the containerization of a multi-tier Java web application by creating Docker images for each microservice, including:
- Load balancer (Nginx)  
- Application server (Tomcat/Java)  
- Database (MariaDB/MySQL)  
- Caching engine (Memcached)  
- Message queue (RabbitMQ)  

The application is deployed as containers and orchestrated with Docker Compose.

## Project Setup
### 1. Clone the project repository
```bash
git clone -b main git clone https://github.com/devopshydclub/vprofile-project.git
cd vprofile-project
```

### 2. Build the artifact
```bash
sudo apt install maven
mvn install
```

## 3. Create Docker Images for Microservices
### 3.1 Application Server (Tomcat)
- Use a Tomcat image matching your Java version (Java 11).  
- Dockerfile:
```dockerfile
FROM tomcat:8-jre11
RUN rm -rf /usr/local/tomcat/webapps/*
COPY target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
WORKDIR /usr/local/tomcat/
CMD ["catalina.sh", "run"]
```
- Build the image:
```bash
docker build -t app:v1 .
```

### 3.2 Database (MySQL)
- Use MySQL image (version 5.7.25).  
- Dockerfile:
```dockerfile
FROM mysql:5.7.25
ENV MYSQL_ROOT_PASSWORD="admin123"
ENV MYSQL_DATABASE="accounts"
ADD db_backup.sql docker-entrypoint-initdb.d/db_backup.sql
```
- Copy the backup SQL:
```bash
cp /home/vagrant/vprofile-project/src/main/resources/db_backup.sql .
```
- Build the image:
```bash
docker build -t db:v1 .
```

### 3.3 Nginx Load Balancer
- Use the latest Nginx image.  
- Create configuration file `nginx.conf`:
```nginx
upstream vproapp {
    server app01:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://vproapp;
    }
}
```
- Dockerfile:
```dockerfile
FROM nginx
RUN rm -rf /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/vproapp.conf
```
- Build the image:
```bash
docker build -t lb:v1 .
```

### 3.4 RabbitMQ
- Use the latest RabbitMQ image:
```bash
docker pull rabbitmq
```

### 3.5 Memcached
- Use the latest Memcached image:
```bash
docker pull memcached
```

## 4. Push Images to Docker Hub
1. Login to Docker Hub:
```bash
docker login -u <your-username>
```
2. Tag images for Docker Hub:
```bash
docker tag <local-image:tag> <username/repo:tag>
```
3. Push images:
```bash
docker push <username/repo:tag>
```

## 5. Docker Compose Setup

Create `docker-compose.yml`:
```yaml
version: "3"
services:
  db01:
    image: akramgalal/node.js:db
    container_name: db01
    ports:
      - "3306:3306"
    volumes:
      - dbdata:/var/lib/mysql
    environment:
      - MYSQL_USERNAME=admin
      - MYSQL_ROOT_PASSWORD=admin123

  mc01:
    image: akramgalal/node.js:mmc
    container_name: mc01
    ports:
      - "11211:11211"
    depends_on:
      - db01

  rmq01:
    image: akramgalal/node.js:rmc
    container_name: rmq01
    ports:
      - "15672:15672"
    environment:
      - RABBITMQ_DEFAULT_USER=test
      - RABBITMQ_DEFAULT_PASS=test

  app01:
    image: akramgalal/node.js:app
    container_name: app01
    ports:
      - "8080:8080"
    volumes:
      - appdata:/usr/local/tomcat/webapps
    depends_on:
      - db01

  lb01:
    image: akramgalal/node.js:lb
    container_name: web01
    ports:
      - "80:80"
    depends_on:
      - app01

volumes:
  dbdata:
  appdata:
```

Start the multi-tier application:
```bash
docker compose up -d
```

## 6. Testing
- Ensure all containers are running:
```bash
docker ps
```
<img width="3359" height="225" alt="Screenshot 2025-09-07 024345" src="https://github.com/user-attachments/assets/d958e380-cb73-47f4-8ffc-b23929d53a8a" />

- Access the application via the load balancer at `http://localhost`

