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

    stage('Set parameter') {
      agent {
        node {
          label 'tools-image'
        }

      }
      steps {
        sh 'properties([[$class: \'JiraProjectProperty\'], [$class: \'BuildConfigProjectProperty\', name: \'\', namespace: \'\', resourceVersion: \'\', uid: \'\'], parameters([string(\'git-url\'), string(defaultValue: \' master\', name: \' git-revision\'), string(defaultValue: \'/source\', name: \'source-dir\'), string(defaultValue: \'""\', name: \'image-url\'), string(defaultValue: \'""\', name: \'app-name\'), string(defaultValue: \'"route"\', name: \'deploy-ingress-type\'), string(description: \'The url of the helm repository\', name: \'helm-curl\')])])'
      }
    }

  }
}