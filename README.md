# Jenkins Multibranch Pipeline with GitHub Webhook

## 1. Create a Multibranch Pipeline in Jenkins

First, create a **Multibranch Pipeline** project in Jenkins.

For example:

```text
Jenkins
   |
   └── Multibranch Pipeline
          |
          └── GitHub Repository
```

The GitHub repository initially has:

```text
main
```

as the default branch.

The Multibranch Pipeline is responsible for automatically discovering branches in the GitHub repository and creating a separate pipeline job for each branch.

---

## 2. Configure the GitHub Repository in Jenkins

In the Multibranch Pipeline:

```text
Branch Sources
      ↓
GitHub / Git
      ↓
Repository URL
      ↓
https://github.com/<username>/<repository>.git
```

Configure **Discover branches** so Jenkins can find branches such as:

```text
main
develop
feature/login
feature/payment
```

Jenkins will create branch-specific jobs automatically.

For example:

```text
Multibranch Pipeline
│
├── main
├── develop
├── feature-login
└── feature-payment
```

---

## 3. Configure the Webhook in Jenkins

For the Multibranch Pipeline, configure the webhook trigger using the **Multibranch Scan Webhook Trigger** plugin.

In Jenkins, configure a token, for example:

```text
Token: multi
```

The token is used to identify which Multibranch Pipeline should respond to the webhook.

The webhook endpoint is:

```text
/multibranch-webhook-trigger/invoke?token=multi
```

---

## 4. Configure the GitHub Webhook

Go to:

```text
GitHub Repository
    ↓
Settings
    ↓
Webhooks
    ↓
Add webhook
```

Configure the Payload URL as:

```text
http://3.89.182.204:8080/multibranch-webhook-trigger/invoke?token=multi
```

Select:

```text
Content type: application/json
```

For events, you can select:

```text
Just the push event
```

or select the events required by your workflow.

---

## 5. How the Webhook Works

Suppose the repository currently has:

```text
main
```

A developer creates a new branch:

```bash
git checkout -b feature/login
```

Then pushes it to GitHub:

```bash
git push origin feature/login
```

The flow is:

```text
Developer
     |
     | git push
     v
GitHub
     |
     | Webhook
     v
Jenkins
     |
     | token=multi
     v
Multibranch Pipeline
     |
     | Scan Repository
     v
New branch detected
     |
     v
feature-login pipeline
```

Jenkins discovers:

```text
feature/login
```

and creates a branch-specific job:

```text
Multibranch Pipeline
│
├── main
└── feature-login
```

---

## 6. How the Pipeline Runs

Jenkins looks for a `Jenkinsfile` in the newly discovered branch.

For example:

```text
feature/login
    |
    └── Jenkinsfile
```

Jenkins checks out the `feature/login` branch and executes its `Jenkinsfile`.

The pipeline could contain:

```text
Checkout
   ↓
Build
   ↓
Unit Test
   ↓
SonarQube
   ↓
Trivy
   ↓
Docker Build
```

---

## 7. Important Point

The **feature branch does not know about Jenkins**.

The branch is simply pushed to GitHub.

The GitHub webhook notifies Jenkins.

The Jenkins **Multibranch Pipeline** discovers the new branch and creates/runs the corresponding branch pipeline.

Therefore:

```text
GitHub Branch
      ↓
GitHub Webhook
      ↓
Jenkins Multibranch Pipeline
      ↓
Branch Discovery
      ↓
Jenkinsfile
      ↓
Pipeline Execution
```

---

## 8. Why We Use a Multibranch Pipeline

We don't need to manually create a separate Jenkins job for every feature branch.

For example, if developers create:

```text
feature/login
feature/payment
feature/search
feature/profile
```

Jenkins automatically discovers them:

```text
Multibranch Pipeline
│
├── main
├── feature-login
├── feature-payment
├── feature-search
└── feature-profile
```

This is the main advantage of a Jenkins **Multibranch Pipeline**.

---

## Interview Answer

> We use a Jenkins Multibranch Pipeline for our GitHub repository. The main branch is initially available in the repository. We configured branch discovery so Jenkins can automatically detect new branches. We also configured a GitHub webhook using the Multibranch Scan Webhook Trigger plugin. When a developer pushes a new branch to GitHub, GitHub sends a webhook request to Jenkins using the configured payload URL and token. Jenkins then scans the repository, discovers the new branch, creates a branch-specific pipeline job, checks out that branch, reads the Jenkinsfile from the branch, and executes the pipeline.

### Important distinction

```text
GitHub
   |
   | Webhook
   v
Jenkins Multibranch Pipeline
   |
   | Branch discovery
   v
feature/login
   |
   | Jenkinsfile
   v
Pipeline execution
```

The **webhook triggers Jenkins to scan/discover changes**, while the **Multibranch Pipeline manages the individual branch pipelines**.
