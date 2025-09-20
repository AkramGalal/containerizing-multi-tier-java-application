# Containerizing Multi-tier Java Web Application
This project demonstrates the containerization of a multi-tier Java web application by creating Docker images for each microservice, including:
- Load balancer (Nginx)  
- Application server (Tomcat/Java)  
- Database (MariaDB/MySQL)  
- Caching engine (Memcached)  
- Message queue (RabbitMQ)  

The application is deployed as containers and orchestrated with Docker Compose.

## Project Setup
### 1. Create Docker Images for Microservices
### 1.1 Application Server (Tomcat)
- Use a Tomcat image matching your Java version (Java 11).  
- Dockerfile:
```dockerfile
FROM openjdk:11 AS build_image
RUN apt update && apt install maven -y
RUN git clone https://github.com/devopshydclub/vprofile-project.git
RUN cd vprofile-project && git checkout docker &&  mvn install 

FROM tomcat:9-jre11
LABEL "Project"="Vprofile"
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=build_image vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
```
- Build the image:
```bash
docker build -t app:v1 .
```

### 1.2 Database (MySQL)
- Use MySQL image (version 8.0.33).  
- Dockerfile:
```dockerfile
FROM mysql:8.0.33
LABEL "Project"="Vprofile"
ENV MYSQL_ROOT_PASSWORD="vprodbpass"
ENV MYSQL_DATABASE="accounts"
ADD db_backup.sql docker-entrypoint-initdb.d/db_backup.sql
```

- Build the image:
```bash
docker build -t db:v1 .
```

### 1.3 Nginx Load Balancer
- Use the latest Nginx image.  
- Create configuration file `nginx.conf`:
```nginx
upstream vproapp {
 server vproapp:8080;
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
LABEL "Project"="Vprofile"
RUN rm -rf /etc/nginx/conf.d/default.conf
COPY nginvproapp.conf /etc/nginx/conf.d/vproapp.conf
```
- Build the image:
```bash
docker build -t lb:v1 .
```

### 1.4 RabbitMQ
- Use the latest RabbitMQ image:
```bash
docker pull rabbitmq
```

### 1.5 Memcached
- Use the latest Memcached image:
```bash
docker pull memcached
```

## 2. Push Images to Docker Hub
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

## 3. Docker Compose Setup
  - Create `docker-compose.yml`
  - Start the multi-tier application:
    ```bash
    docker compose up -d
    ```

## 5. Testing
  - Ensure all containers are running:
  ```bash
  docker ps
  ```
  <img width="3359" height="225" alt="Screenshot 2025-09-07 024345" src="https://github.com/user-attachments/assets/d958e380-cb73-47f4-8ffc-b23929d53a8a" />

  - Access the application via the load balancer at `http://localhost`

