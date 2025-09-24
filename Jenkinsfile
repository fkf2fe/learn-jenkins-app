pipeline {
    agent any

    stages {
        stage('Run NPM with docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            
            steps {
                sh '''
                    npm install
                    npm start
                '''
            }
        }
    }
}