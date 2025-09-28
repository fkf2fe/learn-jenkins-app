pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'ec75396f-cb50-49b0-9ef7-5f0531a02f18'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        stage('Docker') {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci  // clean install-preferred over install in CI environment
                    npm run build
                    ls -la
                '''
            }
        }
        stage("run-tests") {
            parallel {
                stage("Test") {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Test stage"
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage("E2E") {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                            // args '-u root:root' don't
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
  
        stage('Deploy-staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "(undefined)" // assigned by subsequent parsing of deploy-output.json
            }            
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json 
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json) # CAUTION: Assign without spaces
                    echo "run E2E test on ${CI_ENVIRONMENT_URL}"
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }        
        // stage("Approve") {
        //     steps {
        //         timeout(time: 30, unit: 'MINUTES') {
        //             input message: 'Ready to deploy?', cancel: 'NO',  ok: 'YES'
        //         }
        //     }
        // }
        stage('Deploy-prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://fancy-gaufre-21cfae.netlify.app'
            }            
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }
}