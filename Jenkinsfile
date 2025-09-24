pipeline {
    agent any

    stages {
        stage('without docker') {
            steps {
                sh '''
                    echo "Without docker"
                    touch no-docker.txt
                    ls -la
                '''
            }
        }

        stage('Run NPM with docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            
            steps {
                sh '''
                    echo "with docker"
                    npm --version
                    touch container.txt
                    ls -la
                '''
            }
        }
    }
}