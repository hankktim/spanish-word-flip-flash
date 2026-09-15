pipeline {
    agent any

    options {
        ansiColor('xterm')
    }

    stages {
        stage('Install dependencies') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                    reuseNode true
                    args '--ipc=host'
                }
            }
            steps {
                sh 'node --version'
                sh 'npm --version'
                sh 'npm ci'
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                    reuseNode true
                    args '--ipc=host'
                }
            }
            steps {
                sh 'npm run build'
            }
        }

        stage('Test') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                            reuseNode true
                            args '--ipc=host'
                        }
                    }
                    steps {
                        sh 'npx vitest run --reporter=verbose'
                    }
                }

                stage('Integration tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                            reuseNode true
                            args '--ipc=host'
                        }
                    }
                    steps {
                        sh 'npx playwright test'
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'Mock deployment was successful!'
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                    reuseNode true
                    args '--ipc=host'
                }
            }
            environment {
                E2E_BASE_URL = 'https://spanish-cards.netlify.app/'
            }
            steps {
                sh 'npx playwright test'
            }
        }
    }

    post {
        always {
            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'reports-e2e/html',
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report',
                reportTitles: '',
                useWrapperFileDirectly: false
            ])
            archiveArtifacts(
                artifacts: 'reports-e2e/html/**/*',
                allowEmptyArchive: true,
                fingerprint: false
            )
            junit(
                testResults: 'reports-e2e/junit.xml',
                allowEmptyResults: true,
                stdioRetention: 'ALL'
            )   
        }
    }
}