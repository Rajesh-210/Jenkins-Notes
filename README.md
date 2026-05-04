# Jenkins Master-Agent Architecture

## Overview

Jenkins follows a **Master-Agent (Controller-Agent)** architecture to distribute workloads across multiple machines.

* **Master / Controller** → Manages Jenkins
* **Agent / Node** → Executes actual jobs

This architecture improves:

* Scalability
* Performance
* Security
* Parallel execution
* Resource management

---

# Architecture Diagram

```text
Developer Pushes Code
          ↓
     Jenkins Master
          ↓
  Schedules Build Job
          ↓
 Chooses Suitable Agent
          ↓
Agent Executes Build/Test/Deploy
          ↓
Result Sent Back to Master
```

---

# Jenkins Master Responsibilities

The Jenkins Master (Controller) is responsible for:

* Managing Jenkins UI
* Scheduling jobs
* Managing pipelines
* User authentication
* Monitoring agents
* Storing build history
* Assigning jobs to agents

> Best Practice:
> Avoid running heavy builds directly on the master server.

---

# Jenkins Agent Responsibilities

The Jenkins Agent performs actual execution tasks:

* Building applications
* Running tests
* Docker builds
* Deployments
* Executing shell scripts
* Running Maven/Gradle/npm commands

---

# Why Use Master-Agent Architecture?

## Problems Without Agents

If all jobs run on a single Jenkins server:

* High CPU usage
* High memory usage
* Slow builds
* System crashes
* Resource bottlenecks

## Benefits of Agents

* Distributed workload
* Faster CI/CD pipeline
* Parallel job execution
* Environment isolation
* Easy scalability
* Better performance

---

# Real-Time Practical Example

| Machine          | Purpose                       |
| ---------------- | ----------------------------- |
| Jenkins Master   | Job scheduling and management |
| Linux Agent      | Java/Maven builds             |
| Windows Agent    | .NET builds                   |
| Docker Agent     | Container builds              |
| Kubernetes Agent | Deployments                   |

---

# SSH-Based Communication

Jenkins Master usually connects to agents using:

* SSH
* JNLP

Most commonly used method is SSH.

---

# Why SSH?

SSH is preferred because it is:

* Secure
* Standard remote access protocol
* Easy to automate
* Supports passwordless authentication
* Reliable for Linux servers

---

# Practical SSH Flow

```text
Jenkins Master → SSH → Jenkins Agent
```

The master remotely executes commands on the agent:

```bash
ssh ubuntu@agent-ip
```

Example commands executed on agent:

```bash
git clone <repo>
mvn clean package
docker build .
kubectl apply -f deployment.yml
```

---

# Node Configuration in Jenkins

Go to:

```text
Manage Jenkins
→ Nodes
→ New Node
```

Important fields:

| Field                 | Value                          |
| --------------------- | ------------------------------ |
| Host                  | Agent Private IP               |
| Credentials           | Master Machine Private SSH Key |
| Remote Root Directory | /home/ubuntu                   |
| Launch Method         | Launch agents via SSH          |

---

# Why Agent IP?

Because Jenkins Master connects to the Agent machine.

Example:

```text
Master IP : 172.31.10.5
Agent IP  : 172.31.10.20
```

In node configuration, use:

```text
172.31.10.20
```

---

# Why Master Private Key?

The Jenkins Master initiates the SSH connection.

So:

* Master uses its private key
* Agent stores the matching public key in:

```bash
~/.ssh/authorized_keys
```

---

# SSH Key Setup

## Step 1 — Generate Key on Master

```bash
ssh-keygen
```

Files created:

```text
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

---

## Step 2 — Copy Public Key to Agent

```bash
ssh-copy-id ubuntu@agent-ip
```

---

## Step 3 — Configure Node in Jenkins

Use:

* Agent private IP
* Master private SSH key

---

# Interview Explanation

## Short Answer

> Jenkins master acts like a controller and agent acts like a worker node.
> The master manages jobs and agents execute the builds.
> We provide the agent private IP because master connects to the agent through SSH.
> We use the master machine private key for authentication.

---

# One-Line Memory Trick

```text
Master = Manager 🧠
Agent = Worker 🔨
```

---

# Modern Terminology

| Old Term | New Term   |
| -------- | ---------- |
| Master   | Controller |
| Slave    | Agent      |

Modern Jenkins avoids using the term "slave".

---

# Key Interview Points

* Distributed builds
* Parallel execution
* Scalability
* Secure SSH communication
* Environment-specific agents
* Better resource management
* Faster CI/CD pipelines

---

# Conclusion

Jenkins Master-Agent architecture is used to distribute workloads efficiently across multiple machines. The master manages and schedules jobs, while agents perform actual execution tasks such as build, test, and deployment. SSH-based communication provides secure and automated remote execution between the master and agents.

---

# Additional DevOps Interview Topics

# 1. Configuration Management

## Definition

Configuration management is the process of managing and maintaining server configurations automatically.

Instead of manually installing packages and configuring services on multiple servers, automation tools are used.

## Common Tools

* Ansible
* Chef
* Puppet
* SaltStack

## Practical Example

Installing Nginx on 50 servers automatically using Ansible.

## Interview Answer

> Configuration management means maintaining infrastructure in a consistent and automated way using tools like Ansible.

---

# 2. Jenkins Integration

## Definition

Integration means connecting Jenkins with external tools for CI/CD automation.

## Common Integrations

| Tool       | Purpose                |
| ---------- | ---------------------- |
| GitHub     | Source code management |
| Docker     | Containerization       |
| Kubernetes | Deployment             |
| SonarQube  | Code quality analysis  |
| Slack      | Notifications          |

## Example Flow

```text
GitHub → Jenkins → Docker → Kubernetes
```

## Interview Answer

> Jenkins integration means connecting Jenkins with tools like GitHub, Docker, Kubernetes, and SonarQube to automate the software delivery process.

---

# 3. User Management in Jenkins

## Definition

User management controls authentication and authorization inside Jenkins.

## Examples

| Role      | Permissions              |
| --------- | ------------------------ |
| Admin     | Full access              |
| Developer | Build and configure jobs |
| Tester    | View reports             |

## Interview Answer

> Jenkins user management is used to control access and permissions for different users using role-based access control.

---

# 4. Credentials Management

## Definition

Credentials management securely stores secrets inside Jenkins.

## Examples

* GitHub token
* SSH keys
* Docker Hub credentials
* AWS access keys

## Jenkins Path

```text
Manage Jenkins
→ Credentials
```

## Why Important?

* Avoid hardcoding passwords
* Secure deployments
* Secure repository access

## Interview Answer

> Credentials management securely stores passwords, SSH keys, and tokens used in pipelines.

---

# 5. Jenkins Pipeline

## Definition

A Jenkins pipeline is an automated workflow that defines build, test, and deployment stages.

## Pipeline Flow

```text
Build → Test → Deploy
```

## Basic Declarative Pipeline

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building Application'
            }
        }

        stage('Test') {
            steps {
                echo 'Running Tests'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Application'
            }
        }
    }
}
```

---

# 6. Post Failure Trigger in Jenkins

## Example

```groovy
post {
    failure {
        echo 'Build Failed'
    }
}
```

## Use Cases

* Send email notification
* Slack alert
* Stop deployment

## Interview Answer

> Jenkins post-failure actions are used to handle failed stages and send alerts or stop further execution.

---

# 7. Pipeline Models

## Types of Pipelines

| Type                 | Description               |
| -------------------- | ------------------------- |
| Declarative Pipeline | Simple and structured     |
| Scripted Pipeline    | Advanced Groovy scripting |

## Declarative Example

pipeline {
    agent any

    stages {

        stage('Clone Code') {
            steps {
                echo 'Cloning code from GitHub'
            }
        }

        stage('Build') {
            steps {
                echo 'Building application using Maven'
            }
        }

        stage('Test') {
            steps {
                echo 'Running test cases'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application'
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}

How to Explain in Interview 🎤

“This is a Declarative Pipeline.
It uses a structured syntax with stages like Clone, Build, Test, and Deploy.

The ‘agent any’ means the pipeline can run on any available Jenkins agent.

Inside each stage, steps define the actions to execute.

The post section handles success and failure conditions for notifications or alerts.”

## Scripted Example

node {

    stage('Clone Code') {
        echo 'Cloning GitHub repository'
    }

    stage('Build') {
        echo 'Building application'
    }

    stage('Test') {
        echo 'Executing test cases'
    }

    stage('Deploy') {
        echo 'Deploying application'
    }

    try {
        echo 'Pipeline Success'
    }

    catch(Exception e) {
        echo 'Pipeline Failed'
    }
}

How to Explain in Interview 🎤

“This is a Scripted Pipeline.
It uses Groovy-based scripting syntax and provides more flexibility compared to Declarative Pipeline.

The pipeline starts with the node block, which defines the execution environment.

Stages are manually written, and advanced conditions or loops can also be implemented in scripted pipelines.”

## Interview Answer

> Jenkins supports Declarative and Scripted pipelines. “Declarative pipeline is simple and structured, while Scripted pipeline provides advanced flexibility using Groovy scripting.”

---

# 8. GitHub Jenkinsfile Execution

## Flow

```text
GitHub Repository
        ↓
Jenkins pulls Jenkinsfile
        ↓
Pipeline executes
```

## Example Jenkinsfile

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Hello Jenkins'
            }
        }
    }
}
```

## Jenkins Steps

```text
New Item
→ Pipeline
→ Pipeline Script from SCM
→ Git
→ Paste Repository URL
```

## Interview Answer

> Jenkins can automatically execute pipelines stored inside GitHub using a Jenkinsfile.

---

# 9. Find Highest Storage Used Files

## Command

```bash
du -ah / | sort -rh | head -10
```

## Explanation

| Command  | Meaning            |
| -------- | ------------------ |
| du       | Disk usage         |
| sort -rh | Sort largest first |
| head     | Show top results   |

---

# 10. Bash Scripting

## Definition

Bash scripting automates Linux tasks using shell commands.

## Example Script

```bash
#!/bin/bash

echo "Backup Started"

tar -czf backup.tar.gz /data

echo "Backup Completed"
```

#!/bin/bash

echo "Install apache2"

apt install apache2 -y

echo "Start apache"

systemctl start apache2

echo "Show the status"

systemctl status apache2

## Use Cases

* Backups
* Monitoring
* Automation
* Deployment

## Interview Answer

> “Bash scripting is used to automate repetitive Linux administration and deployment tasks.

We write Linux commands inside a script file and execute them together.

It is commonly used for deployments, backups, monitoring, file handling, and automation in DevOps environments.”

---

# 11. File Transfer Between Servers

## Methods

* SCP    = Secure Copy Protocol
Used to:
✅ Copy files securely between servers
Uses SSH internally 🔐

* RSYNC  = Remote Sync
Used to:
✅ Synchronize files/directories
✅ Faster than SCP for large data
Only transfers changed files 🚀

* SSH    = Secure Shell
Used to:
✅ Connect securely to remote server
✅ Execute commands remotely


## Example

```bash
scp file.txt ubuntu@server2:/tmp
```

## Interview Answer

> Files can be securely transferred between servers using SCP or RSYNC over SSH.

---

# 12. Bash Conditional Statements

## Example

```bash
if [ $? -eq 0 ]
then
   echo "Success"
else
   echo "Failed"
fi
```

## Meaning

```text
0 = success
non-zero = failure
```

---

# 13. Ansible Scripting

## Example Playbook

```yaml
---
- hosts: all
  become: yes

  tasks:
    - name: install nginx
      apt:
        name: nginx
        state: present
```

## Interview Answer

> Ansible playbooks automate server configuration and deployment tasks.

---

# 14. Start Service Using Ansible

## Start Service

```yaml
- name: start nginx
  service:
    name: nginx
    state: started
```

## Enable at Boot

```yaml
- name: enable nginx
  service:
    name: nginx
    enabled: yes
```

---

# 15. Default Paths in Ansible

| Item           | Path                     |
| -------------- | ------------------------ |
| ansible.cfg    | /etc/ansible/ansible.cfg |
| Inventory File | /etc/ansible/hosts       |

---

# Important DevOps One-Liners

| Topic       | One-Liner                  |
| ----------- | -------------------------- |
| Jenkins     | Automation server          |
| Pipeline    | Automated workflow         |
| Ansible     | Agentless automation tool  |
| Bash        | Linux automation scripting |
| SCP         | Secure file transfer       |
| Credentials | Secure secret storage      |

---

# Real-Time CI/CD Workflow Example

```text
Developer pushes code
        ↓
GitHub webhook triggers Jenkins
        ↓
Jenkins pipeline starts
        ↓
Agent builds application
        ↓
Docker image created
        ↓
Image pushed to Docker Hub
        ↓
Kubernetes deploys application
```

---

# Final Interview Tips

* Explain practically
* Use real-time examples
* Explain workflow step-by-step
* Mention security and automation benefits
* Use simple language

---

# Final Summary

This document covers Jenkins Master-Agent architecture, pipelines, integrations, credentials management, GitHub integration, Bash scripting, file transfer, Ansible automation, and common DevOps interview topics with practical explanations and examples.
