pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build and push')
    }

    environment {
        // Tagging with build number as requested in POC 2 objectives
        IMAGE_NAME = "mypoc2app"
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-creds' // We will configure this in Jenkins later
        BUILD_TAG = "build-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out branch: ${params.BRANCH_NAME}..."
                // Since this might run in a generic directory, we rely on the SCM checkout
                checkout scm
            }
        }

        stage('Lint & Test') {
            parallel {
                stage('Lint') {
                    steps {
                        echo "Running Checkstyle Linters (Simulated)..."
                        // In a real java project: sh '/usr/local/bin/mvn checkstyle:check'
                        sh 'sleep 3'
                        echo "Linting complete!"
                    }
                }
                stage('Test') {
                    steps {
                        echo "Running Backend Unit Tests..."
                        dir('my-docker-app') {
                            sh '/usr/local/bin/mvn clean test'
                        }
                    }
                }
            }
        }

        stage('Build Java Backend') {
            steps {
                echo "Packaging the Java Backend (skipping tests since they passed)..."
                dir('my-docker-app') {
                    sh '/usr/local/bin/mvn package -DskipTests'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker Image inside poc 2 jenkins folder..."
                dir('my-docker-app') {
                    // Tagging it with the build number as requested
                    sh "docker build -t hitansu/${IMAGE_NAME}:${BUILD_TAG} ."
                    sh "docker tag hitansu/${IMAGE_NAME}:${BUILD_TAG} hitansu/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "Logging into DockerHub and pushing image..."
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin'
                    sh "docker push hitansu/${IMAGE_NAME}:${BUILD_TAG}"
                    sh "docker push hitansu/${IMAGE_NAME}:latest"
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up local docker images to save space..."
            sh "docker rmi hitansu/${IMAGE_NAME}:${BUILD_TAG} || true"
            sh "docker rmi hitansu/${IMAGE_NAME}:latest || true"
        }
        success {
            echo "Successfully pushed hitansu/${IMAGE_NAME}:${BUILD_TAG} to DockerHub!"
        }
        failure {
            echo "Pipeline failed! Please check the logs."
        }
    }
}
