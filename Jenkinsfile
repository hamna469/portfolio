pipeline {
    agent any

    environment {
        REPO_URL      = 'https://github.com/hamna469/portfolio'
        SONARQUBE_ENV = 'SonarQube-Server'
        DOCKER_SERVER = 'ubuntu@ip-172-31-29-171'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: "${REPO_URL}"
            }
        }

        stage('Quality Analysis & Deployment') {
            parallel {
                stage('SonarQube Scan') {
                    steps {
                        script {
                            def scannerHome = tool 'SonarQube Scanner'
                            withSonarQubeEnv("${SONARQUBE_ENV}") {
                                sh """
                                ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=portfolio-cloud \
                                -Dsonar.sources=. \
                                -Dsonar.exclusions=**/*.css,**/*.js,**/*.png,**/*.jpg,node_modules/** \
                                -Dsonar.javascript.node.maxspace=512 \
                                -Dsonar.qualitygate.wait=true
                                """
                            }
                        }
                    }
                }

                stage('Docker Build & Deploy') {
                    steps {
                        sshagent(['ubuntu']) {
                            sh """
                            # Copy files to Target Server
                            scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/

                            # Build and Run on Target Server
                            ssh -o StrictHostKeyChecking=no ${DOCKER_SERVER} "
                                cd /home/ubuntu
                                sudo docker build -t portfolio-app .
                                sudo docker stop portfolio-app || true
                                sudo docker rm portfolio-app || true
                                sudo docker run -d -p 80:80 --name portfolio-app portfolio-app
                            "
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Analysis Completed & App Deployed!"
        }
        failure {
            echo "Pipeline Failed. Check SonarQube or SSH connection."
        }
    }
}