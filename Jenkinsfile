pipeline {
    agent any

    environment {
        IMAGE_NAME = "sagargiragani/java-jenkins-demo"
        DOCKERHUB_CREDS = credentials('dockerhub-credentials')
    }

    stages {
        stage('Build') {
            steps {
                echo "🏗️ Building Docker image..."
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "🔹 Pushing image to Docker Hub..."
                sh '''
                    echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                    docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo "🚀 Deploying Docker container..."
                sh '''
                    docker stop java-jenkins-demo || true
                    docker rm java-jenkins-demo || true
                    docker run -d --name java-jenkins-demo -p 8000:8080 ${IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build, Push & Deploy completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}

