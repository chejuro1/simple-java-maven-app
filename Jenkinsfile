pipeline {
  agent {
    node {
      label 'maven'
    }

  }
  stages {
    stage('Release') {
      steps {
        sh '''ls 
oc project react-intro2
oc start-build simple-java-maven-app  --follow --wait -e OUTPUT_IMAGE=${BUILD_NUMBER}'''
      }
    }

    stage('Docker_lint') {
      agent {
        node {
          label 'nodejs'
        }

      }
      steps {
        sh '''npm install -g dockerlint
dockerlint Dockerfile'''
      }
    }

    stage('Helm') {
      agent {
        node {
          label 'maven'
        }

      }
      steps {
        sh '''ls 

pwd '''
      }
    }

  }
}