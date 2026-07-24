pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')
        NETLIFY_SITE_ID = '8731df66-a269-4605-9e63-1fa64f80d71a'
    }

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
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Tests') {
           parallel {
              stage('Unit Test') {
                        agent {
                            docker {
                               image 'node:18-alpine'
                               reuseNode true
                               }
                        }
                        steps {
                            sh '''
                             echo "Test stage"
                             test -f "build/index.html"
                             npm test
                            '''
                        }
                        post {
                                always {
                                  junit '**/junit.xml'
                                }
                        }
              }
            stage('E2E') {
                                agent {
                                    docker {
                                       image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                                       reuseNode true
                                       }
                                }
                                steps {
                                    sh '''
                                      npm install serve
                                      node_modules/.bin/serve -s build &
                                      sleep 10
                                      npx playwright test --reporter=html
                                    '''
                                }
                                post {
                                        always {
                                          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                                        }
                                }

                            }
           }
        }
        stage('Deploy to stage') {
                            agent {
                            docker {
                                image 'node:18-alpine'
                                reuseNode true
                            }
                            }
                            steps {
                                sh '''
                                    npm install netlify-cli@20.1.1
                                    node_modules/.bin/netlify --version
                                    node_modules/.bin/netlify status
                                    node_modules/.bin/netlify deploy --dir=build
                                '''
                            }
                        }
        stage ('Approval') {
        steps {
           input message: 'Ready for Deploy', ok: 'Yea Im ready to deploy'
           }
           }
        stage('Deploy to prod') {
                    agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                    }
                    steps {
                        sh '''
                            npm install netlify-cli@20.1.1
                            node_modules/.bin/netlify --version
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --prod --dir=build --site=$NETLIFY_SITE_ID
                        '''
                    }
                }
                stage('E2E prod') {
                environment {
                       CI_ENVIRONMENT_URL = "https://bucolic-pudding-5a7ec9.netlify.app"
                    }

                                                agent {
                                                    docker {
                                                       image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                                                       reuseNode true
                                                       }
                                                }
                                                steps {
                                                    sh '''
                                                      npx playwright test --reporter=html
                                                    '''
                                                }
                                                post {
                                                        always {
                                                          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright prod Report', reportTitles: '', useWrapperFileDirectly: true])
                                                        }
                                                }

                                            }

    }
}