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
