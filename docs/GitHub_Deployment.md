# GitHub Deployment

## Overview

This project demonstrates how to integrate a Linux server with GitHub to simplify application deployment and version control. The deployment workflow allows the server to securely retrieve the latest application source code from a GitHub repository using SSH authentication.

Using GitHub as the central source repository provides version control, collaboration, and a repeatable deployment process. Once configured, application updates can be deployed by pulling the latest changes directly from the repository without manually transferring files to the server.

This document focuses exclusively on configuring Git, establishing secure communication with GitHub, cloning repositories, and deploying application updates.

---

## Objectives

The objectives of this project are to:

- Install and configure Git
- Generate an SSH key pair
- Connect the server to GitHub
- Clone a remote repository
- Deploy application updates
- Maintain a secure deployment workflow

---

## Deployment Workflow

```text
Developer
     │
     ▼
GitHub Repository
     │
     ▼
Ubuntu Server
     │
     ▼
Git Pull
     │
     ▼
Updated Application
```

---

## Technologies

| Technology | Purpose |
|------------|---------|
| Git | Version Control System |
| GitHub | Source Code Repository |
| SSH | Secure Authentication |
| Ubuntu | Deployment Server |

---

## Prerequisites

Before beginning this deployment, ensure the following requirements are met:

- Ubuntu server has been configured successfully
- GitHub repository has been created
- SSH access to the server is available

Connect to the server.

```bash
ssh -i cloud-web-key.pem ubuntu@EC2_PUBLIC_IP

# Replace cloud-web-key.pem with the SSH private key.
# Replace EC2_PUBLIC_IP with the server's public IP address.
```

---

> **Best Practice**
>
> Store deployment information such as the GitHub repository URL, branch name, deployment directory, and SSH key location in a text file (for example, `github-deployment.txt`). Keeping deployment information organized simplifies future updates and troubleshooting.

---

## Deployment Guide

The following sections describe each deployment step. Every step includes a brief explanation, the required commands, and verification steps.

---

# Step 1 – Install Git

## Objective

Git is a distributed version control system used to manage application source code. Installing Git on the server allows applications to be cloned, updated, and maintained directly from GitHub.

**Run the following commands inside the Ubuntu server.**

### Install Git

```bash
sudo apt update
sudo apt install git -y
```

### Verify the Installation

```bash
git --version
```

Example output

```text
git version 2.xx.x
```

---

# Step 2 – Configure Git

## Objective

Configure Git with the account information that will be associated with commits made from the server.

**Run the following commands inside the Ubuntu server.**

```bash
git config --global user.name "Your Name"
```

Replace:

- `Your Name` with the preferred Git username.

```bash
git config --global user.email "your_email@example.com"
```

Replace:

- `your_email@example.com` with the GitHub email address.

Verify the configuration.

```bash
git config --list
```

---

# Step 3 – Generate an SSH Key

## Objective

SSH keys provide secure authentication between the Ubuntu server and GitHub without requiring a password.

**Run the following commands inside the Ubuntu server.**

Generate the SSH key.

```bash
ssh-keygen -t ed25519 -C "github-deployment"
```

Press **Enter** to accept the default location.

Display the public key.

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the displayed key.

---

# Step 4 – Configure GitHub

## Objective

Register the SSH public key with GitHub so the server can securely access private repositories.

Log in to GitHub.

Navigate to:

```
Repository
    ↓
Settings
    ↓
Deploy Keys
    ↓
Add Deploy Key
```

Paste the public key copied in the previous step.

Enable:

- Allow write access (optional)

Save the deployment key.

---

# Step 5 – Clone the Repository

## Objective

Clone the GitHub repository to the Ubuntu server. The repository becomes the deployment directory used to host the application.

**Run the following commands inside the Ubuntu server.**

Navigate to the deployment directory.

```bash
cd /var/www
```

Clone the repository.

```bash
git clone git@github.com:username/repository.git
```

Replace:

- `username` with the GitHub username or organization.
- `repository` with the repository name.

Verify.

```bash
ls
```

The repository directory should now exist.

---

# Step 6 – Deploy Application Updates

## Objective

Deploy the latest application version by retrieving updates from the GitHub repository.

Navigate to the repository.

```bash
cd /var/www/repository
```

Replace:

- `repository` with the cloned repository directory.

Retrieve the latest updates.

```bash
git pull origin main
```

Replace:

- `main` with the repository's default branch if different.

---

# Step 7 – Verify the Deployment

## Objective

Verify that the application source code has been updated successfully.

Display the repository status.

```bash
git status
```

Expected output

```text
On branch main

Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

Display the most recent commits.

```bash
git log --oneline -5
```

The latest commits should match the GitHub repository.

---

## Deployment Completed

The Ubuntu server is now connected to GitHub through SSH authentication and is capable of securely retrieving application updates from the remote repository. This workflow provides a simple and reliable deployment strategy while maintaining version control and a centralized source of truth for the application code.
