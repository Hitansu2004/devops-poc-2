pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build and push')
    }

    environment {
        // Tagging with build number as requested in POC 2 objectives
        IMAGE_NAME = "mypoc2app"
        REGISTRY = "localhost:5000" // We are using a local private registry
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
                    sh "/Users/hparichha/.rd/bin/docker build -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_TAG} ."
                    sh "/Users/hparichha/.rd/bin/docker tag ${REGISTRY}/${IMAGE_NAME}:${BUILD_TAG} ${REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Push to Private Registry') {
            steps {
                echo "Pushing image to the Local Private Registry..."
                // Since it is a local internal registry, we don't need credentials!
                sh "/Users/hparichha/.rd/bin/docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_TAG}"
                sh "/Users/hparichha/.rd/bin/docker push ${REGISTRY}/${IMAGE_NAME}:latest"
            }
        }
    }

    post {
        always {
            echo "Cleaning up local docker images to save space..."
            sh "/Users/hparichha/.rd/bin/docker rmi ${REGISTRY}/${IMAGE_NAME}:${BUILD_TAG} || true"
            sh "/Users/hparichha/.rd/bin/docker rmi ${REGISTRY}/${IMAGE_NAME}:latest || true"
        }
        success {
            echo "Successfully pushed ${REGISTRY}/${IMAGE_NAME}:${BUILD_TAG} to Private Registry!"
        }
        failure {
            echo "Pipeline failed! Please check the logs."
        }
    }
}
