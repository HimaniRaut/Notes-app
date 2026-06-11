pipeline {
    agent any

    environment {
        Image_Name = "notes-app:latest"
        Container_name = "notes-app-container"
        port = "9091"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/HimaniRaut/Notes-app.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    echo "Building docker image..."
                    docker build -t ${env.Image_Name} .
                """
            }
        }

        stage('Stop existing container') {
            steps {
                sh """
                    echo "Stopping existing container if any..."
                    docker stop ${env.Container_name} || true
                    docker rm ${env.Container_name} || true
                """
            }
        }

        stage('Start a new Container at ${env.port} port') {
            steps {
                sh """
                    echo "Starting a new Container"
                    docker run -d -p ${env.port}:80 --name ${env.Container_name} ${env.Image_Name}
                """
            }
        }

        stage('Verify the container status') {
            steps {
                sh """
                    echo "Verifying the status of the container"
                    docker ps -a
                    sleep 5
                    curl -s http://127.0.0.1:${env.port} | head -n 20 || true
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful !!'
        }

        failure {
            echo 'Deployment Failed!!'
        }
    }
}
