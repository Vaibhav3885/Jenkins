# Jenkins
### 1. CI/CD Pipelines (High Priority)
📌 What to Prepare:
Jenkins or other CI/CD tools (GitHub Actions, GitLab CI, etc.)

Multi-environment deployment (Dev, QA, Stage, Prod)

Approval gates for production deployments

Integration with security scans and notifications

🎯 Key Interview Questions and Sample Answers
### Q1: Explain how you designed a CI/CD pipeline for microservices.
Sample Answer (based on real-world scenario):

In one of my projects, we had multiple microservices, each in its own Git repo. I used Jenkins for CI/CD.

#### For CI: 
When a developer pushes code to the develop branch, it triggers a Jenkins pipeline that:

Pulls the latest code.

Runs unit tests (e.g., via Maven or Pytest).

Performs SAST using tools like SonarQube.

Builds Docker images and pushes them to ECR.

#### For CD:

On merging into the main branch, we trigger the deployment job:

Pulls Helm charts or Terraform modules.

Deploys to Kubernetes (EKS or AKS) using Helm.

Has manual approval step before Prod.

Canary or Rolling Update strategy was used to reduce risk.

### Q2: What Git workflow did you follow?
Answer:

We followed the Gitflow workflow:

``` feature/* ``` branches for new features.

``` develop ``` branch for integration testing.

``` release/* ``` for staging preparation.

main for production-ready code.

Pull Requests were mandatory with peer review.

Tags were used for versioning artifacts in the CI pipeline.

### Q3: How do you implement approval gates in your pipeline?
Answer:

I implemented manual approval steps in Jenkins using the input step.

After deployment to staging, a notification goes to Slack or email.

An approver (e.g., Dev Manager) must click "Proceed" to deploy to production.

We also implemented GitHub PR approvals before merging to main as a pre-check.

### Q4: How do you manage different environments in your pipeline?
Answer:

We use different config files or Helm value files for each environment.

The Jenkins job uses parameters or environment variables to select the target (Dev, QA, Prod).

The same pipeline template handles all environments using conditionals.

### Q5: What tools did you use in your pipeline for notifications or alerts?
Answer:

Slack/GitHub Notifications via Jenkins plugins or custom scripts.

Email notifications for failures.

We integrated Google Chat for real-time updates on build/deploy status.

Used monitoring tools like Prometheus & Grafana for infra alerts.

### 💡 Best Practices to Mention:
Use parameterized pipelines.

Store secrets in Vault or AWS Secrets Manager, not in Jenkinsfiles.

Keep your pipelines modular and reusable.

Use Docker caching and parallel builds to improve speed.



## Pipeline example

```
pipeline {
  agent any
  parameters {
    choice(name: 'ENV', choices: ['dev', 'qa', 'prod'], description: 'Target environment')
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/your-repo.git'
      }
    }
    stage('Build & Unit Test') {
      steps {
        sh 'mvn clean install'
      }
    }
    stage('Docker Build & Push') {
      steps {
        sh 'docker build -t myapp:${BUILD_NUMBER} .'
        sh 'docker push myapp:${BUILD_NUMBER}'
      }
    }
    stage('Deploy') {
      when {
        expression { params.ENV != 'prod' }
      }
      steps {
        sh "./deploy.sh ${params.ENV}"
      }
    }
    stage('Manual Approval for Prod') {
      when {
        expression { params.ENV == 'prod' }
      }
      steps {
        input message: "Deploy to production?"
        sh "./deploy.sh prod"
      }
    }
  }
}
```
