# 🐳 2-Tier Application Deployment Using Docker
Hello, In this project, you are going to deploy a 2-tier application using Flask as the web server and MySQL as the database. Both services are containerized using Docker, ensuring consistency and easy setup across environments. The Flask app and MySQL database communicate over a private Docker network, and everything is orchestrated using Docker-Compose. This setup is perfect for learning modern DevOps practices with real-world tools like Docker, Dockerfile, and docker-compose.

## 🎯 Learning Objectives
- Docker basics
- Dockerfile usage
- Flask containerization
- MySQL setup
- Docker Compose
- Environment variables
- Persistent storage
- Service networking
- Local deployment

## ⚙️ Prerequisites
- Docker installed on your machine.
- Docker Compose configured properly.
- Git installed for cloning project.
- Code editor like VS Code.
- Basic knowledge of Linux.
- Stable internet connection for downloads.

[AWS-Setup](https://github.com/iam-avinash-jagtap/Linux-Server-Deployment-on-AWS-E2)

_Note:- For this project, I used an Ubuntu-based EC2 instance with a custom key pair for secure access. Although the key pair differs from above setups._

# Step 1:- Install Docker on EC2 Instance
_To perform all Docker operations, you must install Docker and switch to the root user to gain full control over your server._
```bash
sudo su -
apt update -y
apt install -y docker
apt install docker-compose -y
```
Verify Docker is installed:
```bash
docker --version
docker-compose --version
```

---

# Step 2:- Start Docker Service
_Docker has been successfully installed. Start the Docker service and check if it's running._
```bash
systemctl start docker
systemctl enable docker
systemctl status docker
```

---
# Step 3:- Clone the repository 
_Go the GitHub repository and clone your application files from repository._
```bash
git clone https://github.com/iam-avinash-jagtap/2-Tier-Application.git
cd 2-Tier-Application/
```
Now You are in your Application Diretory 
 ---
# Step 4:- Write Dockerfile for you application 
1. Write a Dockerfile to Build the Image of Your Application.
```bash
# Use an official Python runtime as the base image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# install required packages for system
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y gcc default-libmysqlclient-dev pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Copy the requirements file into the container
COPY requirements.txt .

# Install app dependencies
RUN pip install mysqlclient
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Specify the command to run your application
CMD ["python","app.py"]
```
2. Build Dockerfile 
```bash
docker build -t flaskapp .
```
3. Check Your Application Image 
```bash
docker images
```

![Images](https://github.com/iam-avinash-jagtap/2-Tier-Application-Deployment-Using-Docker_Contianer/blob/master/Images/Images.png)

# Step 5:- Write Docker-compose file 
1. To run both containers simultaneously, define the services in a docker-compose.yml file.
```bash
version: "3.8"

services:
  mysql:
    user: "${UID}:${GID}"
    image: mysql:5.7
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: devops
      MYSQL_USER: admin
      MYSQL_PASSWORD: admin
    volumes:
      - ./mysql-data:/var/lib/mysql
      - ./message.sql:/docker-entrypoint-initdb.d/message.sql
    networks:
      - twotier
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-proot"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 60s

  flask-app:
    image: flaskapp:latest
    container_name: flask-app
    ports:
      - "5000:5000"
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: root
      MYSQL_DB: devops
    depends_on:
      - mysql
    networks:
      - twotier
    restart: always
    healthcheck:
      test: ["CMD-SHELL","curl -f http://localhost:5000/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

networks:
  twotier:
```
2. Start the containers using Docker Compose:
   
_Use the following command to build and start all services defined in your docker-compose.yml file:_
```bash
docker-compose up -d 
```
#### Now both of your containers are in a running state..
3. Check the running containers using the following command:
```bash
docker ps
```

![Running-Contianers](https://github.com/iam-avinash-jagtap/2-Tier-Application-Deployment-Using-Docker_Contianer/blob/master/Images/Running%20Contianers.png)

# Step 6:- Test Your Application 

_Your setup is now ready for testing._
1. Open your cloud console.
2. Copy the Public IP of your EC2 instance.
3. Open a new tab in your browser.
4. Paste the IP followed by the port:
   
        http://x.x.x.x:5000
    
![Output](https://github.com/iam-avinash-jagtap/2-Tier-Application-Deployment-Using-Docker_Contianer/blob/master/Images/Output.png)

5. Enter any message in the input box and submit it.
6. Go back to your terminal and access the MySQL container. 
7. Check the database to confirm your message has been stored successfully.

![MYSQL-Contianer](https://github.com/iam-avinash-jagtap/2-Tier-Application-Deployment-Using-Docker_Contianer/blob/master/Images/MYSQL-contianer.png)

# Summary 
This project demonstrates the deployment of a 2-tier web application using Flask and MySQL, fully containerized with Docker and orchestrated using Docker Compose. It offers hands-on experience in building and running multi-container applications, configuring environment variables, managing persistent storage, and setting up container networking. This project is ideal for understanding modern DevOps practices and serves as a strong foundation for deploying real-world applications using containerization technologies.