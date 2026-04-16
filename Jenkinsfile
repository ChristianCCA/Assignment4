pipeline {
    agent any

    environment {
        // EC2 connection
        EC2_USER    = "ubuntu"
        EC2_HOST    = "52.15.124.23"
        CRED_ID     = "ec2-ssh-private-key"

        // App location on EC2
        PROJECT_DIR = "/home/ubuntu/pythonprojects/assignment4"

        // Your GitHub repo
        REPO_URL    = "https://github.com/ChristianCCA/Assignment4.git"

        // Virtual environment folder name
        VENV_NAME   = "venv"
    }

    stages {
        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent([CRED_ID]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            set -e

                            echo "Updating server packages..."
                            sudo apt-get update
                            sudo apt-get install -y python3-venv python3-pip git

                            echo "Stopping old Django server on port 8000..."
                            sudo fuser -k 8000/tcp || true

                            echo "Removing old project..."
                            rm -rf ${PROJECT_DIR}
                            mkdir -p /home/ubuntu/pythonprojects

                            echo "Cloning fresh copy from GitHub..."
                            git clone ${REPO_URL} ${PROJECT_DIR}

                            echo "Entering project directory..."
                            cd ${PROJECT_DIR}

                            echo "Creating virtual environment..."
                            python3 -m venv ${VENV_NAME}
                            . ${VENV_NAME}/bin/activate

                            echo "Upgrading pip..."
                            pip install --upgrade pip

                            echo "Installing dependencies..."
                            if [ -f requirements.txt ]; then
                                pip install -r requirements.txt
                            else
                                pip install django
                            fi

                            echo "Running migrations..."
                            python manage.py makemigrations || true
                            python manage.py migrate --noinput

                            echo "Collecting static files..."
                            python manage.py collectstatic --noinput || true

                            echo "Starting Django server..."
                            BUILD_ID=dontKillMe nohup ${PROJECT_DIR}/${VENV_NAME}/bin/python manage.py runserver 0.0.0.0:8000 > django.log 2>&1 &

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
            echo "SUCCESS: Django app deployed to EC2 at http://52.15.124.23:8000/"
        }
        failure {
            echo "FAILURE: Deployment failed. Check Jenkins console output."
        }
    }
}
