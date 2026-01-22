# CI/CD Pipeline using Jenkins, Ansible, Docker & GitHub Webhooks on AWS EC2

## Author
**Mouli S**

---

## 1. Project Overview

This project demonstrates a **real-world CI/CD pipeline** using **Jenkins, Ansible, Docker, and GitHub Webhooks** deployed on **AWS EC2 instances**.


## Architecture Diagram

<img width="963" height="404" alt="image" src="https://github.com/user-attachments/assets/9b1d239b-8591-488d-bad8-0e8c802cef9d" />




### Core Idea
> Any code change pushed to GitHub should automatically trigger Jenkins, which then uses Ansible to build a Docker image and deploy a container on a remote Docker server.

The entire pipeline is **event-driven**, automated, and production-aligned.

---

## 2. Architecture & Components

### EC2 Instances

#### 1. Jenkins Server (Controller Node)
- Java
- Jenkins
- Ansible
- Git
- Docker CLI (optional, for local testing)

#### 2. Docker Server (Managed Node)
- Docker Engine
- SSH enabled
- Receives application files
- Runs Docker containers

---

### Tools Used
- **GitHub** – Source code management
- **GitHub Webhooks** – Trigger builds on code push
- **Jenkins** – CI/CD automation server
- **Ansible** – Configuration management & deployment
- **Docker** – Containerization
- **AWS EC2** – Infrastructure

---

## 3. High-Level Flow (Understand This First)

```
Developer Laptop
        ↓
Push code to GitHub
        ↓
GitHub Webhook triggers Jenkins
        ↓
Jenkins pulls latest code
        ↓
Jenkins runs Ansible Playbook
        ↓
Ansible connects to Docker EC2 via SSH
        ↓
Docker image is built
        ↓
Old container removed (if any)
        ↓
New container started
```

---

## 4. Initial Setup Explained (With Logic)

### 4.1 Create Two EC2 Instances

- **EC2-1:** Jenkins Server
- **EC2-2:** Docker Server

**Reason:**  
Separation of concerns  
- Jenkins → Orchestration & automation  
- Docker → Application execution  

---

### 4.2 Jenkins EC2 Setup

Installed components:
- Java (required for Jenkins)
- Jenkins
- Ansible
- Git

#### Ansible Inventory
File:
```
/etc/ansible/hosts
```

Added Docker server IP:
```
[dockerservers]
3.89.65.151
```

This tells Ansible **where** to execute deployment tasks.

---

### 4.3 SSH Key-Based Authentication (MOST IMPORTANT)

**Why is this needed?**  
Ansible works over SSH. Jenkins must connect to the Docker EC2 **without passwords**.

#### Steps Performed

1. On Jenkins server (as `jenkins` user):
```
ssh-keygen
```

2. Copy public key:
```
cat ~/.ssh/id_rsa.pub
```

3. Paste into Docker server:
```
~/.ssh/authorized_keys
```

4. Restart SSH on Docker server:
```
sudo systemctl reload ssh
```

#### Result
- Jenkins → Docker EC2 SSH works
- Ansible can execute tasks remotely

---

## 5. Jenkins Job Configuration – What Really Happens

### 5.1 Source Code Management (SCM)

- Jenkins is connected to the GitHub repository
- On every build, Jenkins runs `git clone` / `git pull`
- Workspace location:
```
/var/lib/jenkins/workspace/ansible-jenkins-pipeline
```

Jenkins always works with the **latest code**.

---

### 5.2 GitHub Webhook – The Automation Logic

#### What happens when code is pushed?

1. Developer pushes code to GitHub
2. GitHub sends HTTP POST request to:
```
http://<jenkins-ip>:8080/github-webhook/
```
3. Jenkins GitHub plugin receives the event
4. Repository & branch are validated
5. Jenkins job triggers automatically

**No polling. No manual builds. Pure automation.**

---

## 6. Jenkins Shell Execution – File Transfer Logic

Jenkins build step commands:

```
scp -r /var/lib/jenkins/workspace/ansible-jenkins-pipeline/* root@3.89.65.151:~/project
ansible-playbook /var/lib/jenkins/playbooks/deployment.yaml
```

### What This Does

1. Copies project files from Jenkins → Docker EC2
2. Files are placed at:
```
/home/ubuntu/project
```
3. Ansible playbook is executed

---

## 7. Ansible Playbook Explained

### Playbook Configuration

```yaml
hosts: docker
remote_user: ubuntu
become: true
gather_facts: false
```

**Meaning:**
- `hosts: docker` → Target Docker EC2
- `remote_user: ubuntu` → Login user
- `become: true` → Run tasks with sudo
- `gather_facts: false` → Faster execution

---

### Task 1: Build Docker Image

```yaml
- name: Build Docker image
  docker_image:
    name: lee
    tag: latest
    source: build
    build:
      path: /home/ubuntu/project
```

**What happens:**
- Ansible connects to Docker EC2
- Reads Dockerfile from `/home/ubuntu/project`
- Builds Docker image `lee:latest`

**Equivalent Docker command:**
```
docker build -t lee:latest /home/ubuntu/project
```

---

### Task 2: Run Docker Container

```yaml
- name: Run container
  docker_container:
    name: lee-container
    image: lee:latest
    state: started
    ports:
      - "80:80"
```

**What happens:**
- Creates container if not present
- Starts container if stopped
- Maps EC2 port 80 → Container port 80

**Equivalent Docker command:**
```
docker run -d -p 80:80 --name lee-container lee:latest
```

---

## 8. End-to-End Deployment Flow

```
GitHub Repository
        ↓
Jenkins pulls code
        ↓
Files copied to Docker EC2
        ↓
Ansible playbook runs
        ↓
Docker image built
        ↓
Docker container started
        ↓
Application live on EC2 Public IP (port 80)
```

---

## 9. Why This Pipeline Works Perfectly

- Jenkins handles **CI** (triggering & automation)
- Ansible handles **CD** (deployment & orchestration)
- Docker ensures **consistent runtime**
- GitHub Webhooks remove all manual effort

---

## 10. Final Outcome

✔ Fully automated CI/CD pipeline  
✔ Event-driven deployment  
✔ Production-style architecture  
✔ Beginner-friendly & interview-ready project  

---

**Happy Automating 🚀**
