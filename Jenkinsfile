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
            step {
                sh 'mvn -B -DskipTests test package'
            }
            step {
                sh 'mvn -B -DskipTests package package'
            }
            step { 
                sh 'mvn -B -DskipTests install package'
            }
            step {
                sh 'mvn -B -DskipTests clean package'
            }
        }
    }
}
