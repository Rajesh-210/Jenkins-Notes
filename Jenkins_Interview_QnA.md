# Jenkins CI/CD – Interview Questions & Answers

> 65 questions covering fundamentals, pipeline patterns, troubleshooting, and real-world scenarios.

---

## Table of Contents

1. [Core Concepts](#-core-concepts)
2. [Pipeline & Jenkinsfile](#-pipeline--jenkinsfile)
3. [Plugins & Integrations](#-plugins--integrations)
4. [Security & Access Control](#-security--access-control)
5. [Build Triggers & Scheduling](#-build-triggers--scheduling)
6. [Advanced Pipeline Patterns](#-advanced-pipeline-patterns)
7. [Troubleshooting & Scenarios](#-troubleshooting--scenarios)

---

## 🔵 Core Concepts

---

### Q1. What CI/CD tools have you worked on?

I have worked on **Jenkins** and **GitHub Actions**. I have experience setting up and managing Jenkins pipelines, integrating with Git, and automating build, test, and deployment processes.

---

### Q2. What are the different types of pipelines in Jenkins?

Two main types:

| Type | Description |
|------|-------------|
| **Declarative Pipeline** | Structured, user-friendly syntax |
| **Scripted Pipeline** | Flexible, full Groovy-based control |

---

### Q3. What is the difference between Declarative and Scripted Pipeline?

| Aspect | Declarative | Scripted |
|--------|-------------|----------|
| Syntax | Structured blocks (`pipeline {}`) | Groovy `node {}` block |
| Ease of use | Beginner-friendly | Requires Groovy knowledge |
| Flexibility | Limited but sufficient for most cases | Full programming control |
| Error handling | Built-in `post` directive | Manual `try/catch` blocks |

**Declarative example:**

```groovy
pipeline {
    agent any
    stages {
        stage('Build') { steps { /* build steps */ } }
        stage('Test')  { steps { /* test steps  */ } }
        stage('Deploy'){ steps { /* deploy steps*/ } }
    }
}
```

**Scripted example:**

```groovy
node {
    stage('Build')  { /* build steps  */ }
    stage('Test')   { /* test steps   */ }
    stage('Deploy') { /* deploy steps */ }
}
```

---

### Q4. What is Jenkins architecture?

Jenkins follows a **Master–Agent (Master–Slave)** architecture.

| Component | Responsibility |
|-----------|---------------|
| **Master** | Manages UI, schedules jobs, dispatches builds to agents, monitors agents |
| **Agent (Slave)** | Executes the build tasks assigned by the master |

This allows **distributed builds** and **scalability** — multiple agents can be added to handle more builds simultaneously.

---

### Q5. How do you force Jenkins to run a job on a specific node?

Use the `label` parameter in the `agent` directive:

```groovy
pipeline {
    agent { label 'linux' }
    stages {
        stage('Build') {
            steps { /* build steps */ }
        }
    }
}
```

---

### Q8. What are Jenkins dependencies?

Jenkins dependencies are components it relies on to function:

- **Java** – Jenkins is Java-based and requires a compatible JDK
- **Plugins** – extend Jenkins functionality
- **Build tools** – Maven, Gradle, npm, etc. (installed on agents)

---

### Q9. How do you achieve High Availability in Jenkins?

- Set up a **Master–Agent architecture** with multiple agents
- Use **load balancers** to distribute traffic between multiple Jenkins masters
- Use **shared storage** so all instances have access to the same configurations and artifacts
- Tools like **Jenkins Operations Center** can help manage HA setups

---

### Q10. Which folder should be backed up in Jenkins?

The **Jenkins home directory** contains all critical data:

| OS | Default Path |
|----|-------------|
| Linux | `/var/lib/jenkins` |
| Windows | `C:\Program Files (x86)\Jenkins` |

This directory includes: job configurations, plugins, build history, credentials, and workspace data.

---

### Q11. Can you restore a single job in Jenkins?

Yes. Job configuration files are stored under:

```
/var/lib/jenkins/jobs/<job-name>/config.xml
```

To restore a single job, copy its `config.xml` from a backup back into the `jobs/` directory and reload Jenkins configuration.

---

### Q12. What is config reload in Jenkins?

Config reload allows you to **apply configuration changes without restarting** Jenkins. Useful after editing config files or installing new plugins. Can be triggered via:

```
http://<jenkins-url>/reload
```

---

### Q25. What is a build artifact?

A build artifact is any file **produced during a build** and stored for later use — compiled binaries, JAR/WAR files, test reports, Docker images, etc.

---

### Q26. What is a workspace in Jenkins?

A workspace is a **directory on the Jenkins agent** where source code is checked out and the build process runs. Each job gets its own workspace directory.

---

### Q27. What is an executor in Jenkins?

An executor is a **computational slot on a Jenkins agent** that runs one build at a time. Each agent can have multiple executors configured based on available resources and expected workload.

---

### Q34. What is throttling in Jenkins?

Throttling limits the **number of concurrent builds** running on an agent or across the Jenkins environment. Used to prevent resource contention and keep the system stable. Implemented via the **Throttle Concurrent Builds** plugin.

---

### Q23. What is a Multibranch Pipeline?

A Multibranch Pipeline **automatically discovers and manages branches** in a source code repository. It creates a separate pipeline per branch and triggers builds automatically when changes are detected. Useful for managing feature branches, release branches, and pull requests.

---

### Q24. What is Blue Ocean in Jenkins?

Blue Ocean is a **modern UI for Jenkins** offering:
- Visual pipeline editor
- Improved visualization of pipeline stages and their status
- More intuitive interface compared to the classic Jenkins UI

---

### Q32. What is Pipeline as Code?

Pipeline as code is the practice of **defining CI/CD pipelines in a file** (Jenkinsfile) stored alongside the application source code in version control. This makes pipelines auditable, versionable, and reviewable like any other code.

---

### Q33. What is the difference between Freestyle and Pipeline jobs?

| Aspect | Freestyle Job | Pipeline Job |
|--------|---------------|--------------|
| Definition | Graphical UI | Code (Jenkinsfile) |
| Flexibility | Limited | High — supports complex workflows |
| Version Control | Not easily versioned | Stored in Git |
| Parallel Execution | Requires plugins | Natively supported |
| Error Handling | Limited | `try/catch`, `post` directives |
| Reusability | Low | Shared libraries supported |
| Peer Review | Difficult | Easy — it's just code |

---

## 🟢 Pipeline & Jenkinsfile

---

### Q6. How do you run specific stages in a pipeline?

Use the `when` directive to conditionally run a stage:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { echo "Building..." }
        }
        stage('Test') {
            when { branch 'main' }
            steps { echo "Testing on main only..." }
        }
    }
}
```

---

### Q13. How do you run stages in parallel in Jenkins?

Use the `parallel` directive inside a stage:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { /* build steps */ }
        }
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps { echo "Running unit tests..." }
                }
                stage('Integration Tests') {
                    steps { echo "Running integration tests..." }
                }
            }
        }
        stage('Deploy') {
            steps { /* deploy steps */ }
        }
    }
}
```

---

### Q16. How do you pass parameters to a Jenkins job?

Use the `parameters` directive in a Declarative Pipeline:

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Target environment')
        choice(name: 'REGION', choices: ['us-east-1', 'ap-south-1'], description: 'AWS Region')
    }
    stages {
        stage('Deploy') {
            steps {
                echo "Deploying to ${params.ENVIRONMENT} in ${params.REGION}"
            }
        }
    }
}
```

---

### Q20. How do you run a stage even if the previous stage fails?

**Using `post` directive:**

```groovy
pipeline {
    agent any
    stages {
        stage('Build') { steps { /* build */ } }
        stage('Test')  { steps { /* test  */ } }
    }
    post {
        always {
            echo "This runs regardless of build outcome."
        }
        failure {
            echo "This runs only on failure."
        }
        success {
            echo "This runs only on success."
        }
    }
}
```

**Using `try/catch` in Scripted Pipeline:**

```groovy
node {
    stage('Build') {
        try {
            /* build steps */
        } catch (Exception e) {
            echo "Build failed: ${e.message}"
        }
    }
    stage('Post Actions') {
        echo "Runs regardless of previous outcome."
    }
}
```

---

### Q21. How do you create stage dependency inside a pipeline?

Use `when` with an expression checking the current build result:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { /* build steps */ }
        }
        stage('Test') {
            when {
                expression { return currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps { /* test steps — only if build succeeded */ }
        }
    }
}
```

---

### Q22. What is a Jenkinsfile?

A Jenkinsfile is a **text file containing the Jenkins pipeline definition**, written in Groovy DSL. It implements "pipeline as code" and is typically stored in the root of the project repository so the pipeline is version-controlled alongside the source code.

---

### Q36. What is stash and unstash in Jenkins?

`stash` saves files during one stage; `unstash` retrieves them in a later stage — useful when stages run on different agents.

```groovy
stage('Build') {
    steps {
        sh 'mvn clean package'
        stash name: 'build-artifacts', includes: 'target/*.war'
    }
}
stage('Deploy') {
    agent { label 'deploy-server' }
    steps {
        unstash 'build-artifacts'
        /* deploy the WAR */
    }
}
```

---

### Q37. What is archiveArtifacts in Jenkins?

`archiveArtifacts` saves build outputs (binaries, reports) as part of the Jenkins build record, accessible from the Jenkins UI:

```groovy
stage('Archive') {
    steps {
        archiveArtifacts artifacts: 'target/*.war', fingerprint: true
    }
}
```

---

### Q38. What is retry in Jenkins pipeline?

`retry` automatically re-runs a block of code on failure, useful for flaky tests or transient network errors:

```groovy
stage('Test') {
    steps {
        retry(3) {
            sh 'run-flaky-tests.sh'
        }
    }
}
```

---

### Q39. What is timeout in Jenkins pipeline?

`timeout` aborts a stage if it exceeds a time limit, preventing builds from hanging indefinitely:

```groovy
stage('Long Running Task') {
    steps {
        timeout(time: 30, unit: 'MINUTES') {
            sh './long-running-script.sh'
        }
    }
}
```

---

### Q44. How do you run different stages based on environment (dev, qa, prod)?

Use a `choice` parameter with `when` expressions:

```groovy
pipeline {
    agent any
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'], description: 'Target environment')
    }
    stages {
        stage('Deploy to Dev') {
            when { expression { return params.ENVIRONMENT == 'dev' } }
            steps { echo "Deploying to DEV..." }
        }
        stage('Deploy to QA') {
            when { expression { return params.ENVIRONMENT == 'qa' } }
            steps { echo "Deploying to QA..." }
        }
        stage('Deploy to Prod') {
            when { expression { return params.ENVIRONMENT == 'prod' } }
            steps { echo "Deploying to PROD..." }
        }
    }
}
```

---

### Q49. How do you implement approval before production deployment?

Use the `input` step to pause the pipeline and wait for manual approval:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') { steps { /* build */ } }
        stage('Test')  { steps { /* test  */ } }
        stage('Approval') {
            steps {
                input message: 'Approve deployment to Production?', ok: 'Deploy'
            }
        }
        stage('Deploy to Prod') {
            steps { echo "Deploying to Production..." }
        }
    }
}
```

---

### Q55. How do you reuse pipeline code across multiple projects?

Use **Shared Libraries**. Create a Git repo with reusable functions, then import it in any Jenkinsfile:

**`vars/buildAndDeploy.groovy` (shared library):**

```groovy
def call(String env) {
    sh 'mvn clean install'
    echo "Deploying to ${env}"
}
```

**Jenkinsfile in project:**

```groovy
@Library('my-shared-library') _

pipeline {
    agent any
    stages {
        stage('Build & Deploy') {
            steps {
                buildAndDeploy('production')
            }
        }
    }
}
```

Configure in Jenkins: `Manage Jenkins → System → Global Pipeline Libraries`

---

### Q56. How do you handle multiple repositories in a pipeline?

Use multiple `checkout` steps within a single stage:

```groovy
stage('Checkout All Repos') {
    steps {
        dir('repo1') {
            checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                userRemoteConfigs: [[url: 'https://github.com/org/repo1.git']]])
        }
        dir('repo2') {
            checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                userRemoteConfigs: [[url: 'https://github.com/org/repo2.git']]])
        }
    }
}
```

---

### Q62. Jenkins pipeline failed due to timeout. What will you do?

Combine `retry` and `timeout` to handle transient timeout failures gracefully:

```groovy
stage('Long Running Task') {
    steps {
        retry(3) {
            timeout(time: 30, unit: 'MINUTES') {
                sh './long-running-script.sh'
            }
        }
    }
}
```

---

### Q64. How do you deploy to multiple servers using Jenkins?

Use `parallel` to deploy concurrently:

```groovy
stage('Deploy to All Servers') {
    parallel {
        stage('Server 1') { steps { sh 'deploy.sh server1' } }
        stage('Server 2') { steps { sh 'deploy.sh server2' } }
        stage('Server 3') { steps { sh 'deploy.sh server3' } }
    }
}
```

---

## 🟡 Plugins & Integrations

---

### Q7. What plugins have you used in Jenkins?

| Category | Plugin |
|----------|--------|
| Source Control | Git Plugin, GitHub Plugin |
| Build Tools | Maven Plugin |
| Containers | Docker Plugin, Kubernetes Plugin |
| Notifications | Email Extension Plugin, Slack Notification Plugin |
| Quality | SonarQube Plugin, JUnit Plugin |
| Artifacts | Artifactory Plugin |
| Deployment | Deploy to Container Plugin, Ansible Plugin |
| Security | Credentials Plugin, Role-Based Authorization Strategy Plugin |
| UI | Blue Ocean Plugin, Stage View Plugin |
| Pipeline | Pipeline Plugin, Parameterized Trigger Plugin, Shared Libraries |
| Ops | Build Timeout Plugin, Backup Plugin, Throttle Concurrent Builds Plugin |

---

### Q50. How do you store secrets securely in Jenkins?

Use the **Jenkins Credentials Plugin**:

- Navigate to `Manage Jenkins → Credentials`
- Supported types: Secret Text, Username/Password, SSH Key, Certificate
- Reference credentials by ID in pipelines — secret values are never exposed in logs

```groovy
withCredentials([usernamePassword(credentialsId: 'my-creds',
        usernameVariable: 'USER', passwordVariable: 'PASS')]) {
    sh 'deploy.sh $USER $PASS'
}
```

---

### Q65. What will you do if a Jenkins plugin breaks the system?

1. Navigate to `Manage Jenkins → Plugins` and **disable the problematic plugin**
2. **Restart Jenkins** to apply the change
3. Check `/var/log/jenkins/jenkins.log` for error details
4. **Update or reinstall** the plugin if needed
5. If still broken, restore from a **Jenkins home backup**

---

## 🔴 Security & Access Control

---

### Q28. How do you secure Jenkins?

- Enable **authentication** (Jenkins DB, LDAP, Active Directory)
- Use **Role-Based Access Control** (RBAC plugin) to manage permissions
- Keep Jenkins and plugins **up to date** with security patches
- Use **HTTPS** for all communications
- Store secrets using the **Credentials Plugin** — never hardcode in pipelines
- Audit Jenkins logs regularly for suspicious activity
- Limit the number of users with **admin privileges**
- Disable master-node execution — run builds on **agents only**

---

### Q29. What is credentials management in Jenkins?

Credentials management is the secure storage of sensitive data (usernames, passwords, API keys, SSH keys) used by Jenkins jobs. Jenkins' built-in credentials store:

- Keeps secrets **encrypted at rest**
- Exposes values via `credentialsId` — never prints them in logs
- Supports scoping: System, Global, or per-Folder

---

### Q51. How do you manage access for multiple users in Jenkins?

1. Go to `Manage Jenkins → Configure Global Security` and enable authentication
2. Choose an authentication method: Jenkins user database, LDAP, or Active Directory
3. Install the **Role-Based Authorization Strategy** plugin
4. Create roles (e.g., Developer, Tester, Admin) and assign permissions to each role
5. Assign users to roles based on their responsibilities

---

## 🟠 Build Triggers & Scheduling

---

### Q14. How do you trigger a Jenkins job remotely?

Use the Remote Build Trigger URL with an authentication token:

```
http://<jenkins-url>/job/<job-name>/build?token=<TOKEN>
```

For jobs with parameters:

```
http://<jenkins-url>/job/<job-name>/buildWithParameters?token=<TOKEN>&PARAM=value
```

---

### Q15. How do you trigger Jenkins using a webhook?

**In GitHub:**
- Go to repository `Settings → Webhooks → Add Webhook`
- Payload URL: `http://<jenkins-url>/github-webhook/`
- Content type: `application/json`
- Event: `Push`

**In Jenkins:**
- In the job configuration, enable: `GitHub hook trigger for GITScm polling`

> The `/github-webhook/` endpoint is provided by the Jenkins GitHub Plugin and acts as a dedicated listener for GitHub push events.

---

### Q17. How do you schedule a Jenkins job?

Use **Build Periodically** in job configuration with cron syntax:

| Schedule | Cron Expression |
|----------|----------------|
| Every day at midnight | `H 0 * * *` |
| Every day at 2 AM | `H 2 * * *` |
| Every hour | `H * * * *` |
| Every weekday at 9 AM | `H 9 * * 1-5` |

> `H` (hash) is preferred over `0` — it distributes load by picking a consistent random minute for the job.

---

### Q18. What is Poll SCM in Jenkins?

Poll SCM periodically checks the source code repository for changes and **triggers a build if changes are detected**. Unlike webhooks (push-based), Poll SCM is pull-based — Jenkins asks the repository.

- More resource-intensive than webhooks
- Useful when webhook setup is not possible
- Configure using the same cron syntax as scheduled builds

---

### Q19. How do you create dependencies between Jenkins jobs?

**Option 1 – Build other projects** (UI): In the upstream job's post-build actions, select "Build other projects" and specify the downstream job name.

**Option 2 – Pipeline `build` step:**

```groovy
stage('Trigger Downstream') {
    steps {
        build job: 'downstream-job-name', wait: true
        // wait: false → fire and forget
    }
}
```

**Option 3 – Parameterized Trigger Plugin**: Pass parameters from upstream to downstream jobs.

---

### Q35. What are the different types of build triggers in Jenkins?

| Trigger Type | Description |
|-------------|-------------|
| **SCM Polling** | Periodically checks repo for changes |
| **Webhook** | Repo pushes event notification to Jenkins |
| **Scheduled Build** | Runs at fixed times (cron syntax) |
| **Manual Trigger** | User clicks "Build Now" in Jenkins UI |
| **Upstream/Downstream** | One job triggers another on completion |
| **Remote Trigger** | API call or script triggers via URL + token |

---

### Q46. How do you trigger a job after another job completes successfully?

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'mvn clean install' }
        }
        stage('Trigger Downstream') {
            steps {
                build job: 'deploy-job', wait: true, parameters: [
                    string(name: 'VERSION', value: "${env.BUILD_NUMBER}")
                ]
            }
        }
    }
}
```

---

### Q53. How do you run jobs only at night?

Use **Build Periodically** with a cron expression:

```
H 2 * * *   → runs every night at ~2 AM
H 0 * * *   → runs every night at ~midnight
```

---

### Q57. How do you stop duplicate builds running at the same time?

- **Throttle Concurrent Builds Plugin** – limit concurrent builds per job or globally
- **Lockable Resources Plugin** – lock shared resources so only one build accesses them at a time
- In Declarative Pipeline, use `options { disableConcurrentBuilds() }`

```groovy
pipeline {
    agent any
    options {
        disableConcurrentBuilds()
    }
    stages { /* ... */ }
}
```

---

## 🟣 Advanced Pipeline Patterns

---

### Q47. Jenkins is not triggering automatically on code commit. What will you check?

1. Verify the job is configured to use **SCM Polling** or **Webhook trigger**
2. Check that the correct **repository URL and branch** are specified
3. If using **SCM Polling**, check the polling log under `Git Polling Log` in the job
4. If using **Webhooks**, check GitHub `Settings → Webhooks → Recent Deliveries` for failed POSTs
5. Ensure Jenkins is **reachable from the internet** (public IP, correct security group/firewall rules)
6. Check that the GitHub plugin is installed and the Jenkins URL is correctly set under `Manage Jenkins → System`

---

### Q48. How do you achieve zero-downtime deployment using Jenkins?

Implement a **Blue-Green Deployment** strategy:

1. Maintain two identical environments: **Blue** (live) and **Green** (idle)
2. Deploy the new version to the **Green** environment
3. Run smoke tests against Green
4. Switch the **load balancer** to route traffic to Green
5. Blue becomes idle (available for instant rollback)

Jenkins orchestrates each step, and the load balancer switch is the only momentary action — no downtime.

---

### Q59. How do you handle versioning in Jenkins builds?

Common approaches:

```groovy
environment {
    VERSION = "${env.BUILD_NUMBER}"          // build number as version
    // VERSION = sh(script: 'git describe --tags', returnStdout: true).trim()  // Git tag
}
```

- Use `BUILD_NUMBER` for simple incrementing versions
- Use **Git tags** (`git describe`) for semantic versioning
- Embed the version in artifact names: `app-${VERSION}.war`
- Pass the version as a parameter to downstream deployment jobs

---

### Q63. How do you use dynamic agents in Jenkins?

Dynamic agents are provisioned on-demand using cloud plugins:

- **AWS EC2 Plugin** – spins up EC2 instances as agents
- **Kubernetes Plugin** – creates pods as agents, destroyed after job completes
- **Docker Plugin** – runs builds inside Docker containers

```groovy
pipeline {
    agent {
        kubernetes {
            yaml """
            spec:
              containers:
              - name: maven
                image: maven:3.8-openjdk-17
            """
        }
    }
    stages {
        stage('Build') { steps { sh 'mvn clean install' } }
    }
}
```

Benefit: **cost optimization** — agents only exist while needed.

---

## 🔧 Troubleshooting & Scenarios

---

### Q40. What challenges have you faced in Jenkins?

- Build failures due to environment differences or misconfiguration
- Plugin conflicts after upgrades
- Slow builds due to lack of parallelism or caching
- Managing credentials and secrets securely
- Ensuring high availability and disaster recovery
- Debugging complex multi-stage pipelines

---

### Q41. Your Jenkins build is failing randomly. How do you troubleshoot?

1. Check **build logs** for error messages or stack traces
2. Review recent **code or config changes** that may have introduced the issue
3. Check for **resource constraints** on the agent (CPU, memory, disk)
4. Verify **external dependencies** (databases, APIs) are available
5. If tests are flaky, use `retry()` and investigate the unstable tests
6. Run the build on a **different agent** to rule out agent-specific issues
7. Add extra `echo` statements or `sh 'env'` to gather debug context

---

### Q42. Your Jenkins job is very slow. How do you improve performance?

1. **Parallelize** independent stages using `parallel {}`
2. **Cache** dependencies (Maven `~/.m2`, npm `~/.npm`)
3. Remove **redundant build steps**
4. Use **faster/dedicated agents** with more CPU/memory
5. Implement **incremental builds** — only rebuild changed modules
6. Archive only **necessary artifacts** to reduce I/O

---

### Q43. Disk space in Jenkins server is full. What will you do?

1. Configure jobs to **discard old builds**: `options { buildDiscarder(logRotator(numToKeepStr: '10')) }`
2. Delete old **workspace directories**: `cleanWs()` in pipeline or manually under `/var/lib/jenkins/workspace/`
3. Remove unused **artifacts and logs** from old builds
4. Move the Jenkins home directory to a **larger volume**
5. Use **Nexus/Artifactory** to offload artifact storage from Jenkins

---

### Q45. How do you ensure steps run even if the build fails?

Use the `post { always {} }` block:

```groovy
post {
    always   { echo "Always runs — cleanup, notifications, etc." }
    success  { echo "Runs only on SUCCESS" }
    failure  { echo "Runs only on FAILURE — alert team" }
    unstable { echo "Runs when build is UNSTABLE (test failures)" }
    cleanup  { cleanWs() }
}
```

---

### Q52. Jenkins master crashed. How do you recover?

1. Check logs at `/var/log/jenkins/jenkins.log` to identify the cause
2. If a **plugin** caused it, start Jenkins in **safe mode**: `java -jar jenkins.war --safe`
   - Then disable the problematic plugin from UI and restart normally
3. If a **config** issue, restore `config.xml` from backup
4. If a **resource** issue (disk/memory), free up resources and restart
5. If the instance is unrecoverable, **restore Jenkins home** (`/var/lib/jenkins`) from backup to a new instance

---

### Q54. Build works locally but fails in Jenkins. Why?

Common causes:

| Cause | Fix |
|-------|-----|
| Different tool versions (Java, Maven, Node) | Pin versions on agent, match local |
| Missing environment variables | Add to pipeline `environment {}` block |
| Missing dependencies on agent | Install required packages on agent |
| File permission issues | Check Jenkins user has access to required paths |
| Different build config/params | Align Jenkins job config with local setup |
| Agent has less memory | Increase agent heap or add more RAM |

---

### Q58. How do you configure notifications when a build fails?

**Email (Email Extension Plugin):**

```groovy
post {
    failure {
        mail to: 'team@example.com',
             subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
             body: "Check: ${env.BUILD_URL}"
    }
}
```

**Slack (Slack Notification Plugin):**

```groovy
post {
    failure {
        slackSend channel: '#ci-alerts',
                  color: 'danger',
                  message: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
    }
}
```

---

### Q60. You deployed the wrong version to production. What will you do?

1. **Assess impact** – identify what version was deployed and what changed
2. **Rollback immediately** – revert to the last known good version using your deployment pipeline or manually
3. **Communicate** – notify stakeholders and impacted users
4. **Root cause analysis** – determine how the wrong version was triggered (wrong branch, missing `when` condition, manual error)
5. **Preventive measures** – add manual `input` approval gates, enforce branch-based `when` conditions, and tighten deployment permissions

---

### Q61. How do you run tests faster in Jenkins?

1. **Parallel execution** – split test suite across multiple agents using `parallel {}`
2. **Test sharding** – divide tests into shards run concurrently
3. **Test impact analysis** – only run tests affected by recent changes
4. **Optimize slow tests** – profile and fix tests that take disproportionate time
5. **Use faster test frameworks** – choose tools optimized for speed
6. **Cache dependencies** – avoid re-downloading libraries on every run

---

*Prepared for DevOps Interview Preparation | Jenkins CI/CD*
