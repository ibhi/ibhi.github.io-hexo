pipeline {
    node('ec2-fleet') {
        agent {
            docker { image 'node:8-alpine' }
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
}