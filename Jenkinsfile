pipeline {
    agent any

    environment {
        IMAGE_NAME = "hamzashaukat078/library-web"
        DOCKER_CREDS = "dockerhub-creds"
    }

    stages {

        stage("Docker Build") {
            steps {
                sh "docker build -t $IMAGE_NAME:latest app"
            }
        }

        stage("Docker Push") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDS,
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:latest
                    """
                }
            }
        }

        stage("Kubernetes Deployment") {
            steps {
                sh """
                kubectl apply -f k8s/mongo-deployment.yaml
                kubectl apply -f k8s/mongo-service.yaml
                kubectl apply -f k8s/web-deployment.yaml
                kubectl apply -f k8s/web-service.yaml
                """
            }
        }

        stage("Monitoring") {
            steps {
                sh "echo Prometheus and Grafana will monitor the application"
            }
        }
    }
}
