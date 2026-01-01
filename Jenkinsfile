pipeline {
    agent any

    environment {
        REPO_URL      = 'https://github.com/hamna469/portfolio'
        // Target Server (Jahan website dikhegi)
        DOCKER_SERVER = 'ubuntu@ip-172-31-29-171'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: "${REPO_URL}"
            }
        }

        stage('Docker Build & Deploy') {
            steps {
                // Jenkins me 'ubuntu' ID wale credentials check kar lein
                sshagent(['docker-credentials']) {
                    sh """
                    # 1. Files copy karna
                    scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/

                    # 2. Remote commands run karna
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
        success {
            echo "Deployment Successful! Ab check karein: http://172.31.29.171"
        }
        failure {
            echo "Kuch ghalat hua hai. Logs check karein."
        }
    }
}