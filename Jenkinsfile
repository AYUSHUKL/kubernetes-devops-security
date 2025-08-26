pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
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
      withDockerRegistry([credentialsId:"dockerhub", url:""]){
        sh 'docker build -t shukayu/numeric-app:${GIT_COMMIT} .'
        sh 'docker push shukayu/numeric-app:${GIT_COMMIT}'
    }
    }
}
    stage('k8 deployment') {
  steps {
    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KCFG')]) {
      sh '''
        set -euxo pipefail
        export KUBECONFIG="$KCFG"

        IMAGE="shukayu/numeric-app:${GIT_COMMIT}"

        # Don’t mutate the repo file; substitute on the fly and pipe to kubectl
        sed "s#replace#${IMAGE}#g" k8s_PROD-deployment_service.yaml | kubectl -n prod apply -f -

        kubectl -n prod rollout status deploy/numeric-app --timeout=180s || true
        kubectl -n prod get pods -l app=numeric-app -o wide
      '''
    }
  }
}
 }
}
}
}
