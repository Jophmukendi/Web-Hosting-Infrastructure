# Web Hosting Infrastructure on AWS

This project demonstrates the deployment of a production-ready cloud infrastructure capable of hosting multiple web applications on AWS. The environment combines AWS networking services, Linux system administration, Docker containerization, and web technologies to create a scalable and secure hosting platform.

The deployment is divided into two parts:

- Part 1 – AWS Infrastructure: Provision the cloud networking environment by creating the Virtual Private Cloud (VPC), public and private subnets, Internet Gateway, route tables, security groups, and an Ubuntu EC2 instance.
- Part 2 – Linux Server Configuration and DevOps: Configure the Ubuntu server by installing Docker, creating Docker networking, deploying infrastructure containers (Nginx, MySQL, PHP, Apache, and Django), configuring SSL, DNS, and GitHub deployment.

At the end of this guide, the infrastructure will be capable of hosting multiple web applications behind a single Nginx reverse proxy using Docker containers.

<h1 align="center">Part 1 — AWS Infrastructure</h1>
The first part focuses on building the AWS networking environment. Once completed, the EC2 instance will be ready for installing Docker and deploying web applications.

<h3>Step 1 – Create a Virtual Private Cloud (VPC)</h3>

A Virtual Private Cloud (VPC) provides an isolated network where AWS resources can securely communicate. It serves as the foundation of the infrastructure by defining the IP address range, routing, and network boundaries.

Run the following commands from the local computer (PowerShell, Windows Terminal, Git Bash, or Linux/macOS Terminal) with the AWS CLI installed and configured.

Create a VPC

```bash
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=cloud-web-vpc}]'
```

Example output:
```
VpcId: vpc-0123456789abcdef0
```

Enable DNS Resolution:

```bash
aws ec2 modify-vpc-attribute \
    --vpc-id vpc-xxxxxxxxx \   # Replace vpc-xxxxxxxxx with the VpcId returned by the previous command
    --enable-dns-hostnames "{\"Value\":true}"
```
Verify:
```bash
aws ec2 describe-vpcs
```

### Step 2 – Create an Internet Gateway
An Internet Gateway allows resources inside the VPC to communicate with the public Internet. The gateway will later be associated with the public subnet through a route table.

Create the Internet Gateway
```bash
aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=cloud-web-igw}]'
```

Attach the Internet Gateway to the VPC
```bash
aws ec2 attach-internet-gateway \
    --internet-gateway-id igw-xxxxxxxx \   # Replace igw-xxxxxxxx with the InternetGatewayId returned above.
    --vpc-id vpc-xxxxxxxx                  # Replace vpc-xxxxxxxx with the VpcId created in Step 1.
```
Verify:
```bash
aws ec2 describe-internet-gateways
```

### Step 3 – Create the Public Subnet

The public subnet hosts resources that require Internet access, such as the EC2 web server. Instances launched in this subnet can receive public IP addresses and communicate with external clients.

Create the Public Subnet
```bash
aws ec2 create-subnet \
    --vpc-id vpc-xxxxxxxx \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=public-subnet}]'

# Replace vpc-xxxxxxxx with the VpcId created in Step 1.
```
    
Enable Automatic Public IP Assignment
```bash
aws ec2 modify-subnet-attribute \
    --subnet-id subnet-xxxxxxxxxxx \
    --map-public-ip-on-launch

# Replace subnet-xxxxxxxxxxx with the SubnetId returned above.
```

Verify
```bash
aws ec2 describe-subnets
```

### Step 4 – Create the Private Subnet

The private subnet hosts backend resources that should not be directly accessible from the Internet. This subnet is commonly used for databases, internal APIs, or future application servers.

Create the Private Subnet
```bash
aws ec2 create-subnet \
    --vpc-id vpc-xxxxxxxxx \      # Replace vpc-xxxxxxxxx with the VpcId created in Step 1.
    --cidr-block 10.0.2.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=private-subnet}]'
```

Verify
```bash
aws ec2 describe-subnets
```

### Step 5 – Create the Public Route Table

The route table determines how network traffic is routed within the VPC. This configuration allows the public subnet to send Internet-bound traffic through the Internet Gateway.

Create the Route Table
```bash
aws ec2 create-route-table \
    --vpc-id vpc-xxxxxxxx \    # Replace vpc-xxxxxxxxx with the VpcId created in Step 1.
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=public-route-table}]'
```

Create the Default Route
```bash
aws ec2 create-route \
    --route-table-id rtb-xxxxxxxx \       # Replace rtb-xxxxxxxxx with the RouteTableId returned above.
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-xxxxxxxx    # Replace igw-xxxxxxxxx with the InternetGatewayId created in Step 2
```

Verify
```bash
aws ec2 describe-route-tables
```

Step 6 – Associate the Public Route Table




