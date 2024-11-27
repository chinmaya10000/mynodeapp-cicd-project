pipeline {
    agent any
    
    tools {
        nodejs 'node'
    }

    environment {
        IMAGE_NAME = 'chinmayapradhan/node-app'
        IMAGE_TAG = "1.0-${BUILD_NUMBER}"
        SCANNER_HOME = tool 'sonar-scanner'
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
        stage('SonarQube Analysis (SAST)') {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=mynodeapp-cicd -Dsonar.projectName=mynodeapp-cicd"
                    }
                }
            }
        }
        stage('Quality Gate Check') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        waitForQualityGate abortPipeline: false
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
        stage('Run Retire.js(SCA)') {
            steps {
                script {
                    echo 'Running Retire.js to check for outdated libraries and security issues...'
                    docker.image('node:18-bullseye').inside {
                        dir('app') {
                            sh '''
                                # Install RetireJS in a custom directory
                                npm install -g retire --prefix ./retire_env
                        
                                # Add the custom directory to PATH and run RetireJS
                                export PATH=$PATH:./retire_env/bin
                                retire --path . --outputformat json --outputpath retire.json || true
                            '''
                        }
                    }
                }
            }
        }
        stage('Build Image') {
            steps {
                script {
                    echo 'building the docker image...'
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }
        stage('Image Security Scan') {
            steps {
                script {
                    echo 'Scan image with trivy...'
                    sh "trivy image -f json -o trivy.json --severity HIGH,CRITICAL --exit-code 1 ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'PASSWORD', usernameVariable: 'USER')]) {
                        sh "echo $PASSWORD | docker login -u $USER --password-stdin"
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }
    }
}