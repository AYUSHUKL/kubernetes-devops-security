pipeline {
  agent any

  stages {
    stage('Build Artifact') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
      }
    }

    stage('Unit Test') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Docker build and push') {
      steps {
        withDockerRegistry([credentialsId: "dockerhub", url: ""]) {
          sh 'docker build -t shukayu/numeric-app:${GIT_COMMIT} .'
          sh 'docker push shukayu/numeric-app:${GIT_COMMIT}'
        }
      }
    }

    stage('k8 deployment') {
      steps {
        sh '''#!/bin/bash
  set -euxo pipefail
  IMAGE="shukayu/numeric-app:${GIT_COMMIT}"

  sed "s#replace#${IMAGE}#g" k8s_deployment_service.yamll | kubectl -n prod apply -f -
  kubectl -n prod rollout status deploy/numeric-app --timeout=180s || true
  kubectl -n prod get pods -l app=numeric-app -o wide
'''

      }
    }
  }
}
