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
- Build the image from Dockerfile
 ```bash
 docker build -t app:v1 .
 ```

### 1.2 Database (MySQL)
- Use MySQL image (version 8.0.33).  
- Build the image from Dockerfile
 ```bash
 docker build -t db:v1 .
 ```

### 1.3 Nginx Load Balancer
- Use the latest Nginx image.  
- Create configuration file `nginx.conf`.
- Build the image from Dockerfile
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
  - Start the multi-tier application using the docker compose file.
    ```bash
    docker compose up -d
    ```

## 5. Testing
  - Ensure all containers are running:
    ```bash
    docker ps -a
    ```
    <img width="3345" height="859" alt="Screenshot 2025-09-21 020407" src="https://github.com/user-attachments/assets/76f5f1b9-025c-430e-a397-8d074f705372" />
  
  - Access the application from the localhost IP at `http:192.168.242.130`
    
    <img width="3763" height="1924" alt="Screenshot 2025-09-21 020609" src="https://github.com/user-attachments/assets/b5bb885d-7552-45d8-9587-d8086da30536" />


