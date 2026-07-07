# Web Hosting Infrastructure
A production-ready cloud infrastructure built on Amazon Web Services (AWS) to host multiple web applications using Docker, Nginx, PHP-FPM, Apache, Django, and MySQL.

# Project Overview

This project showcases the deployment of a secure and scalable cloud infrastructure capable of hosting multiple web applications on a single AWS EC2 instance. The environment uses Docker containers to isolate services while Nginx acts as a reverse proxy to route traffic to PHP, Apache Server, and Django applications based on their domains or subdomains.

The infrastructure was designed as a hands-on DevOps project to demonstrate practical experience with AWS networking, Linux administration, Docker, web servers, databases, DNS, SSL, and deployment automation.

The deployment is divided into two parts:

Part 1 – AWS Infrastructure: Provision the VPC, subnets, Internet Gateway, route tables, security groups, and EC2 instance.

Part 2 – Linux Server Configuration & DevOps: Install Docker, deploy containers, configure Nginx, MySQL, SSL, DNS, and GitHub deployment.

# Technology Stack
- Cloud:
    Amazon EC2, Amazon VPC, Route 53, Internet Gateway, Security Groups.
- DevOps:
    Docker, Docker Networking, Git, GitHub.
- Web Technologies: 
    Nginx, PHP-FPM, Apache HTTP Server, Django, Gunicorn, MySQL.
- Networking:
    DNS, Reverse Proxy, HTTPS, SSL/TLS

# Security
The infrastructure follows several security best practices.

- Custom VPC
- Public and Private Subnets
- Security Groups
- SSH Authentication using Key Pairs
- HTTPS using Let's Encrypt
- Reverse Proxy Architecture
- Container Isolation
- Persistent Database Storage

# Deployment Guide

The complete deployment guide is available in:
    [docs/Infrastructure.md](../docs/Infrastructure.md)

The guide covers:

1. AWS Infrastructure
1. Docker Installation
1. Docker Networking
1. MySQL Deployment
1. Nginx Reverse Proxy
1. PHP-FPM
1. Apache
1. Django Applications
1. SSL Configuration
1. Route 53
1. GitHub Deployment
1. Infrastructure Verification

# Future Improvements
- Multi-EC2 for High Availability Deployment
- GitHub Actions CI/CD
- Kubernetes Migration
- Terraform
- AWS Load Balancer
- Auto Scaling
- Centralized Monitoring & Logging

# Author

Joseph Mukendi

Cloud Computing & System Administration 

