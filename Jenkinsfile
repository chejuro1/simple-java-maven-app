pipeline {
    agent {
        docker {
            image 'maven:3-alpine' 
            args '-v /root/.m2:/root/.m2' 
        }
    }
    stages {
        stage('Build') { 
            steps {
                sh 'mvn -B -DskipTests compile package' 
            }
        }
        stage('test') {
            step {
                sh 'mvn -B -DskipTests test package'
            }
        }
        stage('package') {
            step {
                sh 'mvn -B -DskipTests package package'
            }
        }
        stage('Build') {
            step { 
                sh 'mvn -B -DskipTests install package'
            }
        }
        stage('deploy') {
            step {
                sh 'mvn -B -DskipTests deploy package'
            }
        
        
        }
    }
}
