pipeline {
    agent any

    environment {
        REPO_URL      = 'https://github.com/hamna469/portfolio'
        SONARQUBE_ENV = 'SonarQube-Server'
        DOCKER_SERVER = 'ubuntu@ip-172-31-29-171'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // credentialsId remove kar diya agar repo public hai
                git branch: 'master', url: "${REPO_URL}"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQube Scanner'
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=portfolio-cloud \
                        -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Docker Build & Deploy') {
            steps {
                // ID 'ubuntu' use ki hai jo aapke screenshot mein hai
                sshagent(['ubuntu']) {
                    sh """
                    scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/

                    ssh -o StrictHostKeyChecking=no ${DOCKER_SERVER} '
                        cd /home/ubuntu
                        sudo docker build -t portfolio-app .
                        sudo docker stop portfolio-app || true
                        sudo docker rm portfolio-app || true
                        sudo docker run -d -p 80:80 --name portfolio-app portfolio-app
                    '
                    """
                }
            }
        }
    }
}