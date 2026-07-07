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


































