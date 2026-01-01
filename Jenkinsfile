pipeline {
    agent any

    environment {
        REPO_URL      = 'https://github.com/hamna469/portfolio'
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
                // AGAR 'ubuntu' ID nahi mil rahi, toh yahan 'docker-credentials' likh kar dekhein
                sshagent(['ubuntu']) {
                    sh """
                    # 1. Files ko remote server par bhejna
                    scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/

                    # 2. Remote server par Docker commands chalana
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
            echo "Deployment Fast & Successful!"
        }
    }
}