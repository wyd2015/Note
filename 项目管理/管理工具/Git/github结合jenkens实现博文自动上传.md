---
title: 'Jenkins实现自动上传博文'
---
```sh
pipeline {
  # Jenkins在哪里以及如何执行Pipeline或者Pipeline子集
  agent {
    docker { image 'docker.io/openjdk:8u222-jre' }
  }

  stages {
    stage ('Build') {
      echo 'Building ...'
    }
    stage ('Sanity check') {
      steps {
        input "Does the staging environment looks ok?"
      }
    }
    stage ('Test') {
      steps {
        sh 'java -version'
      }
    }
    satge ('Deploy') {
      echo 'Deploying ...'
    }
  }

  # 完成时动作
  post {
    always {
      echo 'This will always run'
    }
    success {
      echo 'This will run only if successful'
    }
    failed {
      echo 'This will run only if failed'
    }
    unstable {
      echo 'This will run only if the run was marked as unstable'
    }
    changed {
      echo 'This will run only if the state of the pipeline has changed.'
      echo 'For example, if the Pipeline was previously failing but is now successful'
    }
  }
}
```