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
                        // Scan ko fast karne ke liye exclusions add kiye hain
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
                // 'ubuntu' ID check karein Jenkins credentials mein
                sshagent(['ubuntu']) {
                    sh """
                    scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/

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

    post {
        failure {
            echo "Kuch ghalat hua hai. Logs check karein."
        }
    }
}