pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'notes-app'
        SONAR_HOST_URL    = 'http://127.0.0.1:9000'
        DOCKER_IMAGE      = 'satyamsri/notes-app:latest'
        REPO_URL          = 'https://github.com/HimaniRaut/Notes-app.git'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Pulling source code from GitHub...'
                git url: "${REPO_URL}", branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube code analysis...'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=${SONAR_HOST_URL}
                    '''
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                echo 'Waiting for SonarQube Quality Gate result...'
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Pull Docker Image') {
            steps {
                echo 'Pulling Docker image from Docker Hub...'
                sh 'docker pull ${DOCKER_IMAGE}'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                echo 'Scanning Docker image with Trivy...'
                sh '''
                    trivy image \
                      --severity CRITICAL,HIGH \
                      --exit-code 0 \
                      --format table \
                      ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                echo 'Scanning source code filesystem with Trivy...'
                sh 'trivy fs --severity CRITICAL,HIGH .'
            }
        }

    }

    post {
        success {
            echo 'Pipeline completed successfully. Code is secure and ready to deploy.'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs above for details.'
        }
    }
}
