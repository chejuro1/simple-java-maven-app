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

    stage('Add Helm Repo') {
      agent {
        node {
          label 'helm'
        }

      }
      steps {
        sh '''

helm repo add simple-java-maven-app https://artifactory-tools.itzroks-662002dv9g-6il6rv-6ccd7f378ae819553d37d5f2ee142bd6-0000.mex01.containers.appdomain.cloud/artifactory/simple-java-maven-app
--username admin --password AP6UcCbxbVCtNoqmBqXzqdhhLnVP7yLp1rZYAN
helm repo update
'''
      }
    }

  }
}