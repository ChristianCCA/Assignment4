pipeline {
    agent any

    environment {
        EC2_USER    = "ubuntu"
        EC2_HOST    = "3.144.229.45"
        CRED_ID     = "ec2-ssh-private-key"

        PROJECT_DIR = "/home/ubuntu/pythonprojects/assignment4"
        REPO_URL    = "https://github.com/ChristianCCA/Assignment4.git"

        IMAGE_NAME     = "assignment4-django"
        CONTAINER_NAME = "assignment4-django-container"
    }

    stages {
        stage('Deploy Dockerized Django App to EC2') {
            steps {
                script {
                    sshagent([CRED_ID]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            set -e

                            echo "Updating server packages..."
                            sudo apt-get update
                            sudo apt-get install -y docker.io git

                            echo "Starting Docker..."
                            sudo systemctl enable docker
                            sudo systemctl start docker

                            echo "Removing old project..."
                            rm -rf ${PROJECT_DIR}
                            mkdir -p /home/ubuntu/pythonprojects

                            echo "Cloning fresh copy from GitHub..."
                            git clone ${REPO_URL} ${PROJECT_DIR}

                            echo "Entering project directory..."
                            cd ${PROJECT_DIR}

                            echo "Stopping old container..."
                            sudo docker stop ${CONTAINER_NAME} || true
                            sudo docker rm ${CONTAINER_NAME} || true

                            echo "Removing old Docker image..."
                            sudo docker rmi ${IMAGE_NAME} || true

                            echo "Building Docker image..."
                            sudo docker build -t ${IMAGE_NAME} .

                            echo "Running Docker container..."
                            sudo docker run -d \\
                                --name ${CONTAINER_NAME} \\
                                -p 8000:8000 \\
                                --restart always \\
                                ${IMAGE_NAME}

                            sleep 3

                            echo "Deployment complete. App should be available at http://${EC2_HOST}:8000/"
                        '
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "SUCCESS: Dockerized Django app deployed to EC2 at http://52.15.124.23:8000/"
        }
        failure {
            echo "FAILURE: Docker deployment failed. Check Jenkins console output."
        }
    }
}
