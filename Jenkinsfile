pipeline {

  agent any

  stages {

    stage('Checkout Source') {
      steps {
        git url:'https://github.com/khultman/microservices-demo.git', branch:'master'
      }
    }
    
    stage('Deploy Gremlin Shop') {
      steps {
        script {
          sh "/usr/local/bin/kubectl apply -f $WORKSPACE/release/kubernetes-manifests.yaml"
        }
      }
    }

  }

}
