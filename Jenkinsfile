pipeline {
    agent any

    environment {
        IMAGE_NAME = "sagargiragani45/java-jenkins-demo"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }

    triggers {
        githubPush()   // 🚀 Auto trigger when webhook fires from GitHub
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔹 Checking out source code..."
                checkout scm
            }
        }

        stage('Build & Test inside Maven Docker') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17'   // ✅ latest working Maven + JDK17
                    args '-v /root/.m2:/root/.m2'           // cache Maven deps
                }
            }
            steps {
                echo "🔹 Building and testing using Maven inside container..."
                sh 'mvn clean package -DskipTests=false'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🔹 Building Docker image..."
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "🔹 Pushing image to Docker Hub..."
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo "🚀 Deploying container..."
                sh '''
                    docker rm -f java-jenkins-demo || true
                    docker run -d --name java-jenkins-demo -p 8080:8080 ${IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully for build #${BUILD_NUMBER}"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}

