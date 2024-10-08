pipeline {
    agent any
    stages {
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
                    #node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                    '''
            }
          
        }

        stage('Run Tests') {
            parallel {
                stage('Unit tests') {
                        agent {
                            docker {
                                image 'node:18-alpine'
                                reuseNode true
                            }
                        }

                        steps {
                            sh '''
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

                stage('E2E') {
                            agent {
                                docker {
                                    image 'mcr.microsoft.com/playwright:v1.46.1-jammy'
                                    reuseNode true
                                }
                            }

                            steps {
                                sh '''
                                    npm install serve
                                    node_modules/.bin/serve -s build &
                                    sleep 10s
                                    npx playwright install
                                    npx playwright test --reporter=line
                                '''
                            }
                            post {
                                always {
                                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                                }
                            }
                    }
            }
        }
    }
}
