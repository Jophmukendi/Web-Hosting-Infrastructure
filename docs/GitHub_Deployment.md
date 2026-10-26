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

## Step 8 – Test the GitHub SSH Connection
Test the SSH connection to confirm that GitHub recognizes the local development computer and that SSH is using the dedicated github_developer key.

Run on the local computer:
```
ssh -T git@github.com
```

A successful connection should return a message similar to:
Hi username! You've successfully authenticated, but GitHub does not provide shell access.

## Step 9 – Create a Local Project Directory
Create a directory where the local Git repositories will be stored. This keeps development projects organized in a dedicated location.

Run on the local computer and navigate to the directory::
```
mkdir -p ~/folder_name
cd ~/folder_name
pwd
```

##Step 10 – Clone the GitHub Repository
Clone the existing GitHub repository to the local development computer. This creates a local working copy connected to the remote GitHub repository.

```
git clone git@github.com:username/repository_name.git

```

Verify its status:

```
git status
```

Expected result:
```
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```

Step 11 – Verify the Remote Repository
Verify that the local repository is connected to the correct GitHub repository. Git normally assigns the name origin to the repository from which the project was cloned.

```bash
git remote -v
```

Expected result:
```
origin  git@github.com:username/repository_name.git (fetch)
origin  git@github.com:username/repository_name.git (push)
```

## Step 12 – Verify the Current Branch
Check which Git branch is currently active. For most repositories, the primary branch is named main.

```bash
git branch --show-current
```

## Step 13 – Modify the Application
Create or modify application files normally on the local computer. Git will detect these changes but will not automatically include them in a commit.

For example, create a file:
```bash
touch file_1.txt
echo "Application test" > file_1.txt
```

Check the changes:
```bash
git status
```

Step 14 – Stage the Changes and Commit the Changes
Add the modified files to Git's staging area. The staging area determines which changes will be included in the next commit.
Create a commit to save the staged changes permanently in the local Git repository.

```bash
git add file_1.txt
git add .
git commit -m "Add file_1.txt"
```

## Step 15 – Synchronize with GitHub
Retrieve any commits that may have been added to GitHub before pushing the local changes. Using --rebase helps keep the commit history linear.

```bash
git pull --rebase origin main
```

Step 16 – Push the Changes to GitHub
Push the local commits to the GitHub repository. This is the step that actually transfers the committed changes from the local computer to GitHub.

```bash
git push origin main
```

## Step 17 – Verify the Push
Verify that the local repository is synchronized with GitHub and that there are no remaining uncommitted changes.
```bash
git status
```




