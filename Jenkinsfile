pipeline {
    agent {
        docker { 
            image 'node:8-alpine'
            label 'ec2-fleet' 
        }
    }
    stages {
        stage('Install') {
            steps {
                sh 'npm install'
            }
        }
        stage('Clean') {
            steps {
                sh 'npm run clean'
            }
        }
        stage('Generate') {
            steps {
                sh 'npm run generate'
            }
        }
    }
}