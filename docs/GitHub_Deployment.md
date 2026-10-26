# GitHub Deployment

## Overview

This guide explains how to integrate a Linux server with GitHub to simplify application deployment and version control. The deployment workflow allows the server to securely retrieve the latest application source code from a GitHub repository using SSH authentication.

Using GitHub as the central source repository provides version control, collaboration, and a repeatable deployment process. Once configured, application updates can be deployed by pulling the latest changes directly from the repository without manually transferring files to the server.

This guide configures both the local development computer and the Ubuntu production server to communicate securely with GitHub.

---

## Deployment Workflow

```text
Local Development Computer
          │
          │ git push
          ▼
     GitHub Repository
          │
          │ git pull
          ▼
     Ubuntu Server

```

The local computer is responsible for developing, testing, committing, and pushing code. The Ubuntu server is responsible for pulling and deploying the code.

---

## Prerequisites

Before beginning this deployment, ensure the following requirements are met:

- Ubuntu server has been configured successfully
- GitHub repository has been created
- SSH access to the server is available

---

# Part 1 – Configure the Local Development Computer

The local computer is the development environment. We will configure Git and SSH so this computer can securely push source code to GitHub.

## Step 1 – Install Git
First, verify that Git is installed on the local computer. Git will track changes and communicate with GitHub.

Run on the local computer:
```bash
git --version
```
Expected result: git version 2.x.x

If Git is not installed on macOS and Homebrew is available:
```bash
brew install git
```

Verify again:
```bash
git --version
```

## Step 2 – Configure Git Identity
Configure the name and email Git will associate with commits created from this computer.

Run on the local computer
```bash
git config --global user.name "User Name"
```

Configure the email:
```bash
git config --global user.email "user@gmail.com"
```

Verify:
```bash
git config --list
```

Expected result: User Name and user@gmail.com

## Step 3 – Check Existing SSH Keys

Before creating a new SSH key, check the .ssh directory to see which keys already exist. This prevents overwriting an existing key and helps us choose a clear name for the new GitHub key.

```bash
ls -la ~/.ssh
```

For example, you might see:
```bash
authorized_keys
id_ed25519
id_ed25519.pub
known_hosts
```

If id_ed25519 already exists, do not overwrite it because it may be used for another server or service.

## Step 4 – Generate a Dedicated GitHub SSH Key
Create a new SSH key specifically for GitHub development access. The -f option allows us to specify the filename instead of using the default id_ed25519.

```bash
ssh-keygen -t ed25519 -C "user@gmail.com" -f ~/.ssh/github_developer

# You can change "github_developer" with your project name.
```

The command creates two files:
```
~/.ssh/github_developer               # Private key. Never share it.
~/.ssh/github_developer.pub           # The public key. It's the one we will add to GitHub.
```

## Step 5 – Add the Key to the SSH Agent
Add the new private key to the SSH agent so the local computer can use it for authentication.

On macOS: 
```bash
ssh-add --apple-use-keychain ~/.ssh/github_developer
```

Verify:
```bash
ssh-add -l
```

The new key should appear in the list.

## Step 6 – Configure SSH for GitHub
Because we're no longer using the default id_ed25519 filename, explicitly tell SSH which key should be used when connecting to GitHub.

Open the SSH configuration:
```bash
nano ~/.ssh/config
```

Add:
```bash
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_developer
    IdentitiesOnly yes
```

Save and exit.

Set appropriate permissions:
```bash
chmod 600 ~/.ssh/config
```

Now when Git runs:
```bash
git push origin main
```

and the remote is:
```
git@github.com:username/repository_name.git
```
SSH knows to use: ~/.ssh/github_developer

## Step 7 – Display the Public Key

Display the public part of the dedicated GitHub key.

```bash
cat ~/.ssh/github_developer.pub
```

Copy the complete output and add it to:
Log in to GitHub.

Navigate to:

```
GitHub
    ↓
Settings
    ↓
SSH and GPG keys
    ↓
New SSH key
```























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
