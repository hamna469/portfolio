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
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=**/*.css,**/*.js,node_modules/** \
                        -Dsonar.language=html
                        """
                    }
                }
            }
        }

        stage('Docker Build & Deploy') {
            steps {
                sshagent(['ubuntu']) {
                    sh """
                    # Transfer files
                    scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/

                    # Remote Deploy
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