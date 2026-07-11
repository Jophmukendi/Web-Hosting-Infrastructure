# Linux Server Configuration

## Overview

This project demonstrates the configuration of an Ubuntu Linux server for hosting modern containerized web applications. The server is prepared as a production-ready application host by installing Docker, configuring container networking, deploying infrastructure services, and creating the foundation required for hosting multiple web applications.

The server configuration follows modern DevOps practices by isolating each service inside its own Docker container. Core infrastructure services—including Nginx, MySQL, PHP-FPM, Apache, and Django—communicate through a dedicated Docker network while remaining independently deployable and maintainable.

This document focuses exclusively on Linux server administration and Docker configuration. It assumes that the AWS infrastructure has already been provisioned and that an Ubuntu EC2 instance is available for configuration.

---

## Objectives

The objectives of this project are to:

- Update and secure the Ubuntu server
- Install Docker Engine
- Configure Docker networking
- Create persistent Docker storage
- Deploy infrastructure containers
- Prepare the server for hosting multiple web applications
- Build a modular and scalable container environment

---

## Architecture

<p align="center">
    <img src="images/aws-infrastructure.png" alt="Linux Server Configuration" width="100%">
</p>

---

## Technologies

| Technology | Purpose |
|------------|---------|
| Ubuntu Server | Linux operating system |
| Docker Engine | Container platform |
| Docker Network | Internal communication between containers |
| Docker Volumes | Persistent application and database storage |
| Nginx | Reverse proxy and web server |
| PHP-FPM | PHP application runtime |
| Apache HTTP Server | Static and legacy web hosting |
| Django | Python web application framework |
| Gunicorn | WSGI application server |
| MySQL | Relational database |

---

## Container Architecture

```text
                    Ubuntu Server
                          │
                    Docker Engine
                          │
                   Docker Bridge Network
                          │
      ┌──────────────┬──────────────┬──────────────┐
      │              │              │              │
  nginx-proxy    php-fpm      apache-web     django-app
                          │
                          │
                     mysql-db
```

---














# Part 2 — Linux Server Configuration and DevOps

After provisioning the AWS infrastructure, the next phase configures the Ubuntu server as a production-ready hosting platform. This section installs Docker, creates the container networking environment, provisions persistent storage, and deploys the MySQL database container that will support multiple web applications.

### Step 10 – Install Docker Engine

Docker is a containerization platform that packages applications and their dependencies into isolated containers. Containerization simplifies application deployment, improves scalability, and allows multiple services to run independently while sharing the same operating system.

Run the following commands inside the EC2 instance after connecting through SSH.

Update the Operating System
```bash
sudo apt update
sudo apt upgrade -y
```

Install Required Packages
```bash
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Add Docker's Official GPG Key
```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Add the Docker Repository
```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \

sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker Engine
```bash
sudo apt update

sudo apt install -y docker-ce docker-ce-cli containerd.io
```

Verify the Installation
```bash
docker --version
```

Example output:

    Docker version 28.x.x, build xxxxxxxxx

Allow the Current User to Run Docker
```bash
sudo usermod -aG docker $USER
```

Apply the new group membership.
```bash
newgrp docker
```

Enable Docker at Startup
```bash
sudo systemctl enable docker

sudo systemctl start docker
```

Verify the Docker service.
```bash
sudo systemctl status docker

sudo docker ps a
```

The service status should display active (running).

Test Docker
```bash
docker run hello-world
```

Expected output
```
Hello from Docker!

This message shows that your Docker installation appears to be working correctly.
```

### Step 11 – Create the Docker Network

Docker networking enables containers to communicate securely without exposing internal services to the Internet. Creating a dedicated bridge network allows containers to communicate using container names instead of IP addresses, making the environment easier to manage and scale.

Run the following commands inside the EC2 instance.

Create the Docker Network
```bash
docker network create web-network       # You can change the network name if you want
```

Verify the Network
```bash
docker network ls
```

Inspect the Network
```bash
docker network inspect web-network
```

The network will not contain any containers. As services are deployed, they will automatically join this network and communicate using Docker's built-in DNS.

### Step 12 – Create Docker Volumes

Docker volumes provide persistent storage that survives container removal or recreation. They are essential for preserving databases and application data across deployments and updates.

Create a Persistent Volume for MySQL
```bash
docker volume create mysql-data    # You can change the vokume name if you want
```

Verify the Volume
```bash
docker volume ls
```

Inspect the Volume
```bash
docker volume inspect mysql-data
```
Note: Docker manages the storage location automatically. No additional configuration is required.

### Step 13 – Deploy the MySQL Database Container

The MySQL container provides centralized database services for all hosted web applications. By storing database files inside a Docker volume, data remains available even if the container is removed or upgraded.

Download the MySQL Image
```bash
docker pull mysql:8.0
```

Verify the Image
```bash
docker images
```

Deploy the MySQL Container
```bash
docker run -d \
    --name mysql-db \
    --network web-network \
    --restart unless-stopped \
    -p 3306:3306 \
    -v mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=RootPassword123! \
    -e MYSQL_DATABASE=portfolio_db \
    -e MYSQL_USER=portfolio_user \
    -e MYSQL_PASSWORD=StrongPassword123! \
    mysql:8.0


#Replace the following values before running the command:

    RootPassword123! → Replace with a strong root password.
    portfolio_db → Replace with the desired database name.
    portfolio_user → Replace with the application database user.
    StrongPassword123! → Replace with a strong password for the database user.
```

Verify the Running Container
```bash
docker ps
```

Example output
```
CONTAINER ID   IMAGE       NAME
xxxxxxxxxxxx   mysql:8.0   mysql-db
```

View the Container Logs
```bash
docker logs mysql-db
```

The log should indicate that the MySQL server has initialized successfully and is ready to accept connections.

Connect to the MySQL Server
```bash
docker exec -it mysql-db mysql -u root -p

Note: Enter the root password configured during deployment.
```

Verify the Databases

    Inside the MySQL console, run:
    
    SHOW DATABASES;

Exit the MySQL console.
```
Exit;
```

### Step 14 – Deploy the Nginx Reverse Proxy

Nginx serves as the entry point for all incoming HTTP and HTTPS requests. As a reverse proxy, it routes client requests to the appropriate backend container based on the requested domain or subdomain, allowing multiple web applications to share a single public IP address.

Download the Nginx Image
```bash
docker pull nginx:latest
```

Create the Required Directories
```bash
sudo mkdir -p /var/www               # All web apps directory
sudo mkdir -p /var/nginx/conf.d      # For Nginx config file
sudo mkdir -p /var/nginx/logs        # for Nginx logs file
```

Deploy the Nginx Container
```bash
docker run -d \
    --name nginx-proxy \
    --network web-network \
    --restart unless-stopped \
    -p 80:80 \
    -p 443:443 \
    -v /var/www:/var/www \
    -v /var/nginx/conf.d:/etc/nginx/conf.d \
    -v /var/nginx/logs:/var/log/nginx \
    nginx:latest
```

Verify the Container
```bash
docker ps
```

Expected output
```
CONTAINER ID     IMAGE          NAME
xxxxxxxxxxxx     mysql:8.0      mysql-db
xxxxxxxxxxxx     nginx:latest   nginx-proxy
```

Verify Nginx
```bash
docker logs nginx-proxy
```

### Step 15 – Deploy the PHP-FPM Container

PHP-FPM (FastCGI Process Manager) processes PHP requests received from Nginx. Separating PHP execution from the web server follows modern DevOps practices and allows the application layer to scale independently.

Download the PHP Image
```bash
docker pull php:8.2-fpm
```

Create the Application Directory
```
sudo mkdir -p /var/www/php-app
```

Deploy the PHP-FPM Container
```bash
docker run -d \
    --name php-fpm \
    --network web-network \
    --restart unless-stopped \
    -v /var/www/php-app:/var/www/php-app \
    -w /var/www/php-app \
    php:8.2-fpm
```

Verify the Container
```bash
docker ps
```

Access the Container
```bash
docker exec -it php-fpm bash
```

Verify PHP.
```bash
php -v
```

Exit the container.
```bash
exit
```

### Step 16 – Deploy the Apache Container
Apache provides an additional web server for hosting legacy applications or static websites that do not require Nginx. Running Apache inside its own container allows different web technologies to coexist while remaining isolated.

Download the Apache Image
```bash
docker pull httpd:latest
```

Create the Apache Website Directory
```bash
sudo mkdir -p /var/www/apache-site
```

Deploy the Apache Container
```bash
docker run -d \
    --name apache-web \
    --network web-network \        # Use the same Network created in Step 11.
    --restart unless-stopped \
    -p 8080:80 \
    -v /var/www/apache-site:/usr/local/apache2/htdocs \
    httpd:latest
```

Verify the Container
```bash
docker ps
```

Test Apache
```bash
curl http://localhost:8080
```

The default Apache page should be returned successfully.

### Step 17 – Deploy the Django Application Container

Django applications run inside isolated Docker containers, allowing each application to be deployed, updated, and managed independently. Nginx forwards requests to the appropriate Django container based on the configured virtual host.

Download the Python Image
```bash
docker pull python:3.12
```

Create the Django Application Directory
```bash
sudo mkdir -p /var/www/django-app
```

Deploy the Django Container
```bash
docker run -dit \
    --name django-app \
    --network web-network \
    --restart unless-stopped \
    -v /var/www/django-app:/app \
    -w /app \
    python:3.12 \
    bash
```

Access the Container
```bash
docker exec -it django-app bash
```

Install Required Packages
```python
pip install django gunicorn mysqlclient
```

Verify the Django Installation
```python
django-admin --version
```

Create a New Django Project (Optional)
```python
django-admin startproject webproject .
```

Exit the Container
```python
exit
```

Verify All Running Containers
```bash
docker ps
```

Expected output
```bash
CONTAINER ID      IMAGE            NAME
xxxxxxxxxxxx      nginx:latest     nginx-proxy
xxxxxxxxxxxx      mysql:8.0        mysql-db
xxxxxxxxxxxx      php:8.2-fpm      php-fpm
xxxxxxxxxxxx      httpd:latest     apache-web
xxxxxxxxxxxx      python:3.12      django-app
```

Verify Docker Networking
```bash
docker network inspect web-network
```

The output should confirm that all containers are connected to the same Docker bridge network.

The application hosting platform has now been deployed successfully. The infrastructure includes a reverse proxy, database server, PHP runtime, Apache web server, and a Django application container, all connected through a dedicated Docker network. This modular architecture simplifies application deployment, maintenance, and scaling while following modern DevOps and containerization practices.

### Step 18 – Configure Nginx Virtual Hosts

Nginx virtual hosts allow multiple web applications to share the same server by routing requests based on the requested domain or subdomain. This configuration enables each application to have its own hostname while using a single Nginx reverse proxy.

Create the Nginx Configuration Directory
```bash
sudo mkdir -p /var/nginx/conf.d
```

Create a Virtual Host for a PHP Application
```bash
sudo nano /var/nginx/conf.d/portfolio.conf        #Replace portfolio with your application name
```

Add the following configuration:
```bash
server {

    listen 80;

    server_name portfolio.example.com;          # Replace portfolio.example.com with the desired domain or subdomain.

    root /var/www/php-app/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {

        include fastcgi_params;

        fastcgi_pass php-fpm:9000;

        fastcgi_index index.php;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

Create a Virtual Host for a Django Application
```bash
sudo nano /var/nginx/conf.d/inventory.conf        # Replace inventory with the application name
```

Add the following configuration:
```bash
server {

    listen 80;

    server_name inventory.example.com;            # Replace inventory.example.com with the desired domain or subdomain.

    location / {

        proxy_pass http://django-app:8000;

        proxy_set_header Host $host;

        proxy_set_header X-Real-IP $remote_addr;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Reload the Nginx Container
```bash
docker exec nginx-proxy nginx -t
```

```bash
docker exec nginx-proxy nginx -s reload
```

### Step 19 – Configure SSL Certificates (Let's Encrypt)

SSL certificates encrypt communication between clients and the web server. Using Let's Encrypt provides trusted SSL/TLS certificates at no cost, allowing applications to be securely accessed over HTTPS.

Install Certbot
```bash
sudo apt install certbot -y
```

Generate an SSL Certificate
```bash
sudo certbot certonly \
    --standalone \
    -d portfolio.example.com    # Replace portfolio.example.com with the domain configured in Step 18.
```

Note: Repeat the process (Generate an SSL Certificate) for each hosted domain or subdomain.

Mount the Certificates into the Nginx Container
```bash
docker stop nginx-proxy

docker rm nginx-proxy
```

Redeploy the container.
```bash
docker run -d \
    --name nginx-proxy \
    --network web-network \
    --restart unless-stopped \
    -p 80:80 \
    -p 443:443 \
    -v /var/www:/var/www \
    -v /var/nginx/conf.d:/etc/nginx/conf.d \
    -v /etc/letsencrypt:/etc/letsencrypt \
    nginx:latest
```

Verify HTTPS
```bash
curl https://portfolio.example.com
```

Verify DNS Resolution
```bash
nslookup portfolio.example.com
```

or
```bash
dig portfolio.example.com
```

The command should return the EC2 public IP address.

### Step 21 – Configure GitHub Deployment
GitHub provides version control for the application source code. After changes are pushed to GitHub from the development environment, the EC2 server retrieves the latest version using Git, simplifying deployments while maintaining a central source repository.

Install Git
```bash
sudo apt install git -y
```

Generate an SSH Key
```bash
ssh-keygen -t ed25519 -C "github-deployment"
```

Press Enter to accept the default file location.

Display the Public Key
```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the displayed key and add it to the GitHub repository as a Deploy Key.

Clone the Repository
```bash
cd /var/www
```
```bash
git clone git@github.com:username/repository.git
```
Replace:
- username with the GitHub username or organization.
- repository with the repository name.

Deploy Updates

Whenever changes are pushed to GitHub, update the application.

```bash
cd /var/www/repository
```

```bash
git pull origin main
```

Replace:
- repository with the cloned repository name
- main with the repository's default branch if different.

### Step 22 – Verify the Infrastructure

The final step verifies that the cloud infrastructure, Docker containers, networking, and hosted applications are functioning correctly. Performing these validation checks confirms that the environment is ready for production deployments.

Run the following commands inside the EC2 instance.

Verify Running Containers: 
```bash
docker ps 
```

Verify Docker Networks
```bash
docker network ls
```

Inspect the Docker Network
```bash
docker network inspect web-network
```

All application containers should be attached to the web-network bridge network.

Verify Docker Volumes
```bash
docker volume ls
```

The mysql-data volume should be listed.

Verify Nginx Configuration
```bash
docker exec nginx-proxy nginx -t
```

The output should indicate:
```bash
syntax is ok
test is successful
```
Verify MySQL
```bash
docker exec -it mysql-db mysql -u root -p
```

Verify the Hosted Applications. Open a web browser and access:
```bash
http://portfolio.example.com
https://portfolio.example.com

http://inventory.example.com
https://inventory.example.com
```

Each application should load successfully.































