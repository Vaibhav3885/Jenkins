# Jenkins


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
