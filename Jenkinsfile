pipeline {
    agent {
        docker { 
            image 'node:8-alpine'
            label 'ec2-fleet' 
        }
    }
    
    stages {
        stage('Clean') {
            steps {
                sh 'npm run clean'
            }
        }
    }
    stages {
        stage('Generate') {
            steps {
                sh 'npm run generate'
            }
        }
    }
    
}