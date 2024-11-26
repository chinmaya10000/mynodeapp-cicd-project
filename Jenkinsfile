pipeline {
    agent any
    
    tools {
        nodejs 'node'
    }

    environment {
        IMAGE_NAME = 'chinmayapradhan/node-app'
        IMAGE_TAG = "1.0-${BUILD_NUMBER}"
        SEMGREP_RULES = 'p/javascript'
    }

    stages {
        stage('Clone Repo') {
            steps {
                script {
                    echo 'Pull the latest code'
                    git branch: 'main', url: 'https://github.com/chinmaya10000/mynodeapp-cicd-project.git'
                }
            }
        }

        stage('Secret Scanning with Gitleaks') {
            steps {
                script {
                    echo 'Run Gitleaks scan'
                    sh 'gitleaks detect --source=. -v --report-path=gitleaks-report.json || true'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    dir('app') {
                        sh 'npm install'
                    }
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    dir('app') {
                        sh 'npm test'
                    }
                }
            }
        }
        stage('Run Semgrep') {
            steps {
                script {
                    echo 'Running Semgrep to check for security vulnerabilities in code...'
                    docker.image('returntocorp/semgrep').inside {
                        dir('app') {
                            sh 'semgrep ci --json --output semgrep.json'
                        }
                    }
                }
            }
        }
        stage('Run Retire.js') {
            steps {
                script {
                    echo 'Running Retire.js to check for outdated libraries and security issues...'
                    docker.image('node:18-bullseye').inside {
                        dir('app') {
                            sh 'npm install -g retire'
                            sh 'retire --path . --outputformat json --outputpath retire.json || true'
                        }
                    }
                }
            }
        }
    }
}