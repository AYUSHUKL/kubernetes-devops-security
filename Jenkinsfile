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
    stage('k8 deployment){
  steps{
    withKubeConfig([credentialsID: 'kubeconfig']){
      sh "sed -i 's#replace#shukayu/numeric-app:${GIT_COMMIT}#g' k8s_PROD-deployment_service.yaml"
      sh "kubectl -n prod apply -f k8s_PROD-deployment_service.yaml"
    }
  }
}
}
}
