# Jenkins – Complete DevOps Course Notes

> Course covering Jenkins CI/CD from installation to advanced pipeline patterns.  
> Dates covered: **2 Apr 2026 – 1 May 2026**

---

## Table of Contents

1. [What is Jenkins?](#1-what-is-jenkins)
2. [Installation on Amazon Linux / RHEL](#2-installation-on-amazon-linux--rhel)
3. [Accessing Jenkins](#3-accessing-jenkins)
4. [Jenkins Architecture](#4-jenkins-architecture)
5. [Job Types](#5-job-types)
6. [Jenkins Pipeline](#6-jenkins-pipeline)
7. [Jenkinsfile Examples](#7-jenkinsfile-examples)
8. [Jenkins Upgrade & Backup](#8-jenkins-upgrade--backup)
9. [Distributed Architecture (Master–Agent)](#9-distributed-architecture-masteragent)
10. [Jenkins Restarts](#10-jenkins-restarts)
11. [Parallel Stage Execution](#11-parallel-stage-execution)
12. [Build Continues on Stage Failure](#12-build-continues-on-stage-failure)
13. [Environment Variables](#13-environment-variables)
14. [Webhook – GitHub → Jenkins](#14-webhook--github--jenkins)
15. [Nexus Artifactory](#15-nexus-artifactory)
16. [Maven](#16-maven)
17. [Jenkins Shared Libraries](#17-jenkins-shared-libraries)
18. [Caching in Jenkins](#18-caching-in-jenkins)
19. [When Conditions in Pipeline](#19-when-conditions-in-pipeline)
20. [Tomcat Deployment Notes](#20-tomcat-deployment-notes)
21. [Important File Locations](#21-important-file-locations)

---

## 1. What is Jenkins?

Jenkins is an **open-source automation tool** used for CI/CD (Continuous Integration & Continuous Deployment).

It automates:
- **Build** – compile source code
- **Test** – run automated tests
- **Deploy** – push to servers / containers

**Real DevOps Flow:**

```
GitHub → Jenkins → Build → Test → Deploy
```

---

## 2. Installation on Amazon Linux / RHEL

```bash
# (Optional) Increase /tmp space
mount -o remount,size=4G /tmp

# Update system
sudo yum update -y

# Add Jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/rpm-stable/jenkins.repo

# Import Jenkins GPG key
sudo rpm --import https://pkg.jenkins.io/rpm-stable/jenkins.io-2026.key

# Upgrade packages
sudo yum upgrade -y

# Install Java (required by Jenkins)
sudo yum install java-21-amazon-corretto -y

# Install Jenkins
sudo yum install jenkins -y

# Enable and start Jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

> **Tip:** Save the above as a shell script (`jenkins_setup.sh`) for repeatable installs.

---

## 3. Accessing Jenkins

```
http://<EC2-PUBLIC-IP>:8080
```

Get the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## 4. Jenkins Architecture

| Component | Role |
|-----------|------|
| **Master (Controller)** | Manages UI, schedules jobs, stores config |
| **Agent (Slave/Node)** | Executes jobs and builds |

One Master → Multiple Agents

---

## 5. Job Types

| Type | Description |
|------|-------------|
| **Freestyle** | UI-based, simple jobs |
| **Pipeline** | Code-based CI/CD using Jenkinsfile |
| **Multibranch Pipeline** | Auto-detects branches, creates jobs per branch |

---

## 6. Jenkins Pipeline

### Why Pipeline over Freestyle?

| Freestyle | Pipeline |
|-----------|----------|
| Manual UI config | Code-based (Pipeline as Code) |
| Hard to track changes | Version-controlled in Git |
| Not reusable | Reusable across projects |
| No parallel execution | Supports parallel stages |

### Two Pipeline Types

**1. Declarative Pipeline** – Structured, recommended

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo "Building app"
            }
        }
        stage('Test') {
            steps {
                echo "Testing app"
            }
        }
    }
}
```

**2. Scripted Pipeline** – Flexible, full Groovy

```groovy
node {
    stage('Build') {
        echo "Building app"
    }
    stage('Test') {
        echo "Testing app"
    }
}
```

---

## 7. Jenkinsfile Examples

### Full Pipeline with Maven + Tomcat Deploy

```groovy
pipeline {
    agent any

    environment {
        GIT_URL    = 'https://github.com/devopstraininghub/mindcircuit17d.git'
        GIT_BRANCH = 'main'
        MAVEN_CMD  = 'mvn clean install'
        TOMCAT_URL = 'http://<TOMCAT-IP>:8081/'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_URL}"
            }
        }

        stage('Build') {
            steps {
                echo "Build number: ${env.BUILD_NUMBER}"
                sh "${MAVEN_CMD}"
            }
        }

        stage('Deploy') {
            steps {
                deploy adapters: [
                    tomcat9(
                        credentialsId: 'tomcat',
                        url: "${TOMCAT_URL}"
                    )
                ],
                contextPath: 'myapp',
                war: '**/*.war'
            }
        }
    }
}
```

---

## 8. Jenkins Upgrade & Backup

> **Rule:** Always backup before upgrading.

### Step 1 – Stop Jenkins & Backup WAR

```bash
systemctl stop jenkins

sudo cp -rp /usr/share/java/jenkins.war \
    /usr/share/java/jenkins.war.bak_$(date +%d%b%Y)

cd /usr/share/java/
rm -rf jenkins.war
```

### Step 2 – Backup Jenkins Home

```bash
sudo cp -r /var/lib/jenkins /var/lib/jenkins_backup
```

Jenkins home contains: jobs, configs, plugins, credentials.

### Step 3 – Download New WAR

```bash
cd /usr/share/java/
sudo wget https://get.jenkins.io/war/2.559/jenkins.war
```

### Step 4 – Restart & Verify

```bash
sudo systemctl restart jenkins
sudo systemctl status jenkins
```

Login to UI → Check version in dashboard → Verify jobs work.

### Rollback

```bash
sudo systemctl stop jenkins
sudo mv /usr/share/java/jenkins.war.bak_<date> /usr/share/java/jenkins.war
sudo systemctl start jenkins
```

### Best Practices

- Always test upgrade in **staging** first
- Check **plugin compatibility** before upgrading
- Monitor logs after upgrade: `/var/log/jenkins/jenkins.log`

---

## 9. Distributed Architecture (Master–Agent)

### Why Distributed?

> Building on the built-in node is a **security concern**. Always set up distributed builds.

| Problem (Single Server) | Solution (Distributed) |
|-------------------------|------------------------|
| Heavy load, slow builds | Distribute across agents |
| Resource limitations | Better utilization |
| Security risks | Isolated execution |

### Flow

```
Developer pushes code
        ↓
Jenkins Master receives trigger
        ↓
Master schedules job
        ↓
Job sent to Agent
        ↓
Agent executes build / test / deploy
        ↓
Result returned to Master
```

### Agent Types

| Type | Description |
|------|-------------|
| **Static** | Manually created, always available |
| **Dynamic** | Created on-demand (AWS, Kubernetes) |

### Setting Up an Agent

1. **Manage Jenkins → Nodes → Add New Node**
2. Configure: node name, remote directory, labels, launch method (SSH / JNLP)
3. Connect the agent machine

### Labels

Labels route jobs to specific agents:

```groovy
agent {
    label "docker"
}
```

### Real-Time Example

```
Agent 1  →  Java + Maven
Agent 2  →  Docker
Agent 3  →  Testing tools
```

Jenkins routes each job to the correct agent automatically.

---

## 10. Jenkins Restarts

**Via CLI:**

```bash
systemctl start jenkins
systemctl stop jenkins
systemctl restart jenkins
systemctl status jenkins
```

**Via Browser (GUI):**

```
http://<IP>:8080/restart        # Immediate restart
http://<IP>:8080/safeRestart    # Wait for running jobs to finish
```

---

## 11. Parallel Stage Execution

Run independent stages simultaneously to speed up pipelines:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/devopstraininghub/mindcircuit17d.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Deploy to Multiple Environments') {
            parallel {
                stage('Deploy to LAB') {
                    steps {
                        deploy adapters: [tomcat9(credentialsId: 'tomcat', url: 'http://<IP>:8081/')],
                               contextPath: 'LAB-MC-APP', war: '**/*.war'
                    }
                }
                stage('Deploy to UAT') {
                    steps {
                        deploy adapters: [tomcat9(credentialsId: 'tomcat', url: 'http://<IP>:8081/')],
                               contextPath: 'UAT-MC-APP', war: '**/*.war'
                    }
                }
                stage('Deploy to SBOX') {
                    steps {
                        deploy adapters: [tomcat9(credentialsId: 'tomcat', url: 'http://<IP>:8081/')],
                               contextPath: 'SBOX-MC-APP', war: '**/*.war'
                    }
                }
            }
        }
    }
}
```

---

## 12. Build Continues on Stage Failure

Use `try/catch` blocks so the pipeline continues even if a stage fails:

```groovy
pipeline {
    agent any

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Code') {
            steps {
                script {
                    try {
                        git branch: 'master', url: 'https://github.com/devopstraininghub/mindcircuit17d.git'
                    } catch (Exception e) {
                        echo "Clone failed: ${e.message}"
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    try {
                        sh 'mvn clean install'
                    } catch (Exception e) {
                        echo "Build failed: ${e.message}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying artifact to Tomcat"
            }
        }
    }
}
```

---

## 13. Environment Variables

### Built-in Variables

| Variable | Description |
|----------|-------------|
| `BUILD_NUMBER` | Auto-incremented build number |
| `WORKSPACE` | Path to job's workspace directory |
| `JOB_NAME` | Name of the Jenkins job |
| `BUILD_ID` | Unique ID for the current build |
| `NODE_NAME` | Jenkins agent running the job |
| `GIT_BRANCH` | Git branch (when job is connected to Git) |

### Usage in Pipeline

```groovy
stage('Show Env Variables') {
    steps {
        echo "Job Name: ${env.JOB_NAME}"
        echo "Build Number: ${env.BUILD_NUMBER}"
        echo "Workspace: ${env.WORKSPACE}"
    }
}
```

### Custom Environment Variables

```groovy
pipeline {
    agent any

    environment {
        GIT_URL    = 'https://github.com/org/repo.git'
        GIT_BRANCH = 'main'
        MAVEN_CMD  = 'mvn clean install'
    }

    stages {
        stage('Build') {
            steps {
                sh "${MAVEN_CMD}"
            }
        }
    }
}
```

---

## 14. Webhook – GitHub → Jenkins

### What is a Webhook?

A webhook is an HTTP mechanism that lets GitHub **push events** to Jenkins in real time, instead of Jenkins polling GitHub repeatedly.

### Without vs With Webhook

| Without Webhook | With Webhook |
|-----------------|-------------|
| Jenkins polls GitHub constantly | GitHub notifies Jenkins instantly |
| Wasted resources | Real-time trigger |
| Delayed builds | Faster builds |

### Flow

```
Developer pushes code
        ↓
GitHub triggers webhook (HTTP POST)
        ↓
Jenkins receives at /github-webhook/
        ↓
Jenkins triggers the matching job
```

### Configuration

**In Jenkins:**
- Enable: `GitHub hook trigger for GITScm polling`

**In GitHub:**
- **Payload URL:** `http://<jenkins-url>/github-webhook/`
- **Content type:** `application/json`
- **Event:** `Push`

> The `/github-webhook/` endpoint is provided by the Jenkins GitHub Plugin. Jobs will not trigger if this path is missing.

---

## 15. Nexus Artifactory

Nexus by Sonatype is an **artifact repository manager**.

| Property | Value |
|----------|-------|
| Vendor | Sonatype |
| Default Port | **8081** |
| Language | Java |
| License | Open Source |

### Installation (Quick Steps)

```bash
sudo yum install java-17-openjdk -y

cd /app
sudo wget -O nexus.tar.gz https://download.sonatype.com/nexus/3/nexus-3.91.1-04-linux-x86_64.tar.gz
tar -xvf nexus.tar.gz
mv nexus-3.91.1-04/ nexus

sudo adduser nexus
sudo chown -R nexus:nexus /app/nexus
sudo chown -R nexus:nexus /app/sonatype-work

# Set run_as_user in nexus.rc
sudo vi /app/nexus/bin/nexus.rc
# Add: run_as_user="nexus"

# Create systemd service
sudo vi /etc/systemd/system/nexus.service
sudo systemctl enable nexus
sudo systemctl start nexus

# Get admin password
cat /app/sonatype-work/nexus3/admin.password
```

### Integrating Nexus with Maven

**In `pom.xml`:**

```xml
<distributionManagement>
    <snapshotRepository>
        <id>NexusRepo</id>
        <url>http://<NEXUS-IP>:8081/repository/my-repo-snapshot/</url>
    </snapshotRepository>
    <repository>
        <id>NexusRepo</id>
        <url>http://<NEXUS-IP>:8081/repository/my-repo-release/</url>
    </repository>
</distributionManagement>
```

**In `/etc/maven/settings.xml` on Jenkins/Maven server:**

```xml
<server>
    <id>NexusRepo</id>
    <username>admin</username>
    <password>your-password</password>
</server>
```

### Snapshot vs Release

| Version | Meaning |
|---------|---------|
| `1.0.0` | Stable release |
| `1.0.0-SNAPSHOT` | In development / ongoing |

---

## 16. Maven

### What is Maven?

Maven is a **build automation and dependency management tool** for Java projects.

It automates:

```
Source Code → Compile → Test → Package (JAR/WAR) → Deploy
```

### Maven Project Structure

```
project/
├── src/
│   ├── main/java/
│   ├── main/resources/
│   └── test/java/
├── pom.xml
└── target/        ← build output
```

### pom.xml – Key Elements

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>       <!-- org/company name -->
    <artifactId>my-app</artifactId>      <!-- project name -->
    <version>1.0</version>
    <packaging>war</packaging>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
        </dependency>
    </dependencies>
</project>
```

### Maven Lifecycle

```
validate → compile → test → package → verify → install → deploy
```

### Common Commands

| Command | Description |
|---------|-------------|
| `mvn clean` | Remove old build files from `target/` |
| `mvn compile` | Compile source code |
| `mvn test` | Run unit tests |
| `mvn package` | Create JAR/WAR |
| `mvn install` | Install artifact to local repo (`~/.m2`) |
| `mvn clean install` | Full clean build |
| `mvn clean package` | Clean then package |
| `mvn clean deploy` | Build and push to remote repository |

### Maven Repositories

| Type | Location |
|------|----------|
| Local | `~/.m2/repository` |
| Central | `https://repo.maven.apache.org` |
| Remote | Custom (e.g., Nexus) |

---

## 17. Jenkins Shared Libraries

### What is a Shared Library?

Reusable Jenkins pipeline code stored in one Git repository, shared across multiple jobs and teams.

### Problem → Solution

```
10 projects × same build steps = 10 duplicate pipelines  ❌
          ↓
Write once → Use everywhere                               ✅
```

### Shared Library Structure

```
(Git Repo)
vars/
    build.groovy        ← define reusable functions here
src/
    (advanced Groovy classes)
resources/
    (template files)
```

### Example

**`vars/build.groovy`:**

```groovy
def call() {
    echo "Building application..."
    sh 'mvn clean install'
}
```

**`Jenkinsfile` in project:**

```groovy
@Library('my-shared-lib') _

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                build()
            }
        }
    }
}
```

### Configuration in Jenkins

`Manage Jenkins → System → Global Pipeline Libraries`

- **Name:** `my-shared-lib`
- **Source:** Git repo URL

### Benefits

- Code reuse across teams
- Single place to update pipeline logic
- Standardized pipelines
- Faster development

---

## 18. Caching in Jenkins

### What is Caching?

Storing previously downloaded/built data so it can be reused in future builds — avoiding redundant work.

### Maven Cache

Maven automatically caches dependencies in `~/.m2/repository`.

```
First build  → downloads dependencies → stores in ~/.m2
Next builds  → reads from ~/.m2 → no re-download  🚀
```

### Docker Layer Cache

Docker reuses unchanged layers on every `docker build`.  
Only modified layers are rebuilt.

### NPM Cache

```bash
npm install   # caches packages in ~/.npm
```

### Impact

| | First Build | Subsequent Builds |
|-|-------------|-------------------|
| Time | ~10 mins | ~2 mins |
| Reason | Downloads everything | Uses cache |

### Best Practices

- Use **persistent workspace** (same agent/node)
- Avoid deleting cache directories unnecessarily
- Clear cache only when it becomes corrupted

---

## 19. When Conditions in Pipeline

### What is `when`?

Controls whether a stage **runs or is skipped** based on a condition.

### Syntax

```groovy
stage('Deploy') {
    when {
        <condition>
    }
    steps {
        // runs only if condition is true
    }
}
```

### Common Conditions

**Branch-based:**

```groovy
when {
    branch 'main'
}
```

**Environment variable:**

```groovy
when {
    environment name: 'ENV', value: 'prod'
}
```

**Custom expression:**

```groovy
when {
    expression {
        return env.BUILD_NUMBER.toInteger() > 5
    }
}
```

**File change:**

```groovy
when {
    changeset "**/*.java"   // any Java file changed anywhere
}
```

**Tag-based:**

```groovy
when {
    tag "v*"
}
```

### Real-Time Use Case

```
Build  → always run
Test   → always run
Deploy → only when branch = main  (prevent accidental deploys)
```

### Notes

- Works in **Declarative Pipeline** only
- Evaluated **before** the stage executes
- Stage is skipped (not failed) if condition is false

---

## 20. Tomcat Deployment Notes

### Normal WAR Deployment

```
app.war  →  accessible at  http://server:8080/app
```

### Deploy as Root Application

Rename the WAR to `ROOT.war` → accessible at `http://server:8080/`

Tomcat auto-deploys when `autoDeploy="true"` in `server.xml`:

```xml
<Host autoDeploy="true" unpackWARs="true">
```

### Clean Deployment Steps

```bash
# Stop Tomcat
./bin/shutdown.sh

# Remove old ROOT
rm -rf webapps/ROOT/
rm -rf webapps/ROOT.war

# Copy new WAR
cp new-app.war webapps/ROOT.war

# Start Tomcat
./bin/startup.sh
```

### Tomcat Users Config (`conf/tomcat-users.xml`)

```xml
<?xml version="1.0" encoding="utf-8"?>
<tomcat-users>
    <role rolename="manager-gui"/>
    <user username="tomcat" password="tomcat"
          roles="manager-gui, manager-script, manager-status"/>
</tomcat-users>
```

---

## 21. Important File Locations

| File / Directory | Purpose |
|------------------|---------|
| `/usr/share/java/jenkins.war` | Jenkins core application |
| `/var/lib/jenkins/` | Jenkins home (jobs, plugins, credentials) |
| `/var/log/jenkins/jenkins.log` | Jenkins logs |
| `/var/lib/jenkins/secrets/initialAdminPassword` | First-time login password |
| `~/.m2/repository` | Maven local dependency cache |
| `/etc/maven/settings.xml` | Maven global settings (Nexus credentials) |
| `/app/sonatype-work/nexus3/admin.password` | Nexus initial admin password |

---

## Quick Reference – Key Commands

```bash
# Jenkins service
systemctl start|stop|restart|status jenkins

# Jenkins web restarts
http://<IP>:8080/restart
http://<IP>:8080/safeRestart

# Maven build
mvn clean install
mvn clean package
mvn clean deploy

# SCP file to remote server
scp -i key.pem file.txt ec2-user@<IP>:/tmp/

# sed – replace text in files
sed 's/old/new/g' input.txt > output.txt
sed -i 's/old/new/g' file.txt   # in-place edit
```

---

*Notes compiled from DevOps Training – Batch 17D | Apr–May 2026*
