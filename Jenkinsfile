pipeline {
  agent any
  environment {
    IMAGE = "nourheneouertani/fleetman-queue"
  }
  stages {
    stage('Build') {
      steps { sh 'docker build -t $IMAGE:$BUILD_NUMBER .' }
    }
    stage('Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
          sh 'docker push $IMAGE:$BUILD_NUMBER'
        }
      }
    }
    stage('Deploy') {
      steps {
        sh 'kubectl set image deployment/queue queue=$IMAGE:$BUILD_NUMBER'
        sh 'kubectl rollout status deployment/queue --timeout=300s'
        sh 'kubectl rollout restart deployment/position-simulator deployment/position-tracker'
        sh 'kubectl rollout status deployment/position-simulator --timeout=300s'
      }
    }
  }
  post {
    always { sh 'docker image prune -f' }
  }
}
