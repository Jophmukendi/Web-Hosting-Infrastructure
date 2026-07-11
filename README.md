# Cloud Web Hosting Platform on AWS

A production-ready cloud infrastructure built on **Amazon Web Services (AWS)** for hosting multiple web applications using Docker containers. The platform combines AWS networking, Linux system administration, containerization, and modern web technologies to provide a scalable, secure, and maintainable hosting environment.

---

## Project Overview

This project demonstrates the design and implementation of a complete cloud-based web hosting platform capable of hosting multiple PHP and Django applications behind a single Nginx reverse proxy.

The infrastructure follows a layered architecture:

- **Cloud Infrastructure** provisions the networking and compute resources using AWS.
- **Linux Server Configuration** prepares the Ubuntu server and Docker environment.
- **GitHub Deployment** provides version control and a deployment workflow for application updates.

---

## Technology Stack

### Cloud

- Amazon EC2
- Amazon VPC
- Public & Private Subnets
- Internet Gateway
- Route Tables
- Security Groups
- Amazon Route 53

### Operating System

- Ubuntu Server LTS

### DevOps

- Docker
- Docker Networking
- Docker Volumes
- Git
- GitHub

### Web Technologies

- Nginx
- PHP-FPM
- Apache HTTP Server
- Django
- Gunicorn
- MySQL

---

## Architecture

<p align="center">
    <img src="images/cloud-topology.png" alt="Cloud Web Hosting Platform" width="100%">
</p>

The platform consists of:

- AWS Virtual Private Cloud (VPC)
- Ubuntu EC2 Server
- Docker Engine
- Docker Bridge Network
- Nginx Reverse Proxy
- PHP-FPM Container
- Apache Container
- Multiple Django Containers
- MySQL Database Container
- Route 53 DNS
- GitHub Deployment Workflow

---

## Documentation

The project documentation is organized into independent implementation guides.

| Document | Description |
|----------|-------------|
| 📘 **[AWS Infrastructure](docs/AWS_Infrastructure.md)** | Deploy the AWS networking environment, including the VPC, subnets, Internet Gateway, Security Groups, and EC2 instance. |
| 🐳 **[Linux Server Configuration](docs/Linux_Server_Configuration.md)** | Configure the Ubuntu server, install Docker, create Docker networking, and deploy the infrastructure containers. |
| 🚀 **[GitHub Deployment](docs/GitHub_Deployment.md)** | Configure GitHub integration, SSH authentication, repository cloning, and application deployment. |

---

## Repository Structure

```text
Web-Hosting-Infrastructure/
│
├── README.md
│
├── docs/
│   ├── AWS_Infrastructure.md
│   ├── Linux_Server_Configuration.md
│   ├── GitHub_Deployment.md
│   └── images/
│
├── nginx/
├── mysql/
├── php/
├── django/
└── scripts/
```

---

## Skills Demonstrated

- AWS Cloud Infrastructure
- Linux System Administration
- Docker Containerization
- Reverse Proxy Configuration
- Network Design
- Database Administration
- Git & GitHub
- Infrastructure Documentation
- DevOps Best Practices

---

## Future Enhancements

- GitHub Actions CI/CD
- HTTPS Automation
- Monitoring & Alerting
- Infrastructure as Code (Terraform)
- Kubernetes Deployment
- AWS Load Balancer
- Auto Scaling

---

## License

This project is licensed under the **MIT License**.
