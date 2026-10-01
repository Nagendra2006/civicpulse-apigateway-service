pipeline {
    agent any

    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Building GatewayService..."
                    ./mvnw clean compile
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Running GatewayService tests..."
                    ./mvnw test
                '''
            }
        }

        stage('Secret Scan') {
            steps {
                sh '''
                    echo "Running Trivy secret scan..."

                    trivy fs \
                      --scanners secret \
                      --skip-dirs target \
                      --exit-code 1 \
                      .
                '''
            }
        }

        stage('Verify Docker') {
            steps {
                sh '''
                    echo "Docker location:"
                    which docker

                    echo "Docker version:"
                    docker --version
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building GatewayService Docker image..."

                    docker build \
                      -t civicpulse-gateway-service:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh '''
                    echo "Scanning GatewayService Docker image..."

                    trivy image \
                      --timeout 15m \
                      --severity HIGH,CRITICAL \
                      civicpulse-gateway-service:${BUILD_NUMBER}
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-civicpulse',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                          -u "$DOCKER_USERNAME" \
                          --password-stdin

                        docker tag \
                          civicpulse-gateway-service:${BUILD_NUMBER} \
                          ${DOCKER_USERNAME}/civicpulse-gateway-service:${BUILD_NUMBER}

                        docker push \
                          ${DOCKER_USERNAME}/civicpulse-gateway-service:${BUILD_NUMBER}

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'GatewayService CI pipeline completed successfully.'
        }

        failure {
            echo 'GatewayService CI pipeline failed.'
        }

        always {
            echo "Build Number: ${BUILD_NUMBER}"
        }
    }
}