pipeline {
    agent any

    environment {
        IMAGE_NAME   = "umer055/my-web-app"
        DOCKER_CREDS = "dockerhub-creds"
    }

    stages {

        /* ============================
           1. CODE FETCH STAGE
           ============================ */
        stage("Code Fetch Stage") {
            steps {
                echo "Source code fetched automatically from GitHub using SCM"
            }
        }

        /* ============================
           2. DOCKER IMAGE CREATION
           ============================ */
        stage("Docker Image Creation Stage") {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:latest app
                """
            }
        }

        /* ============================
           3. DOCKER IMAGE PUSH
           ============================ */
        stage("Docker Image Push Stage") {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: DOCKER_CREDS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        /* ============================
           4. KUBERNETES DEPLOYMENT
           ============================ */
        stage("Kubernetes Deployment Stage") {
            steps {
                sh """
                kubectl apply --validate=false -f k8s/mongo-deployment.yaml
                kubectl apply --validate=false -f k8s/mongo-service.yaml
                kubectl apply --validate=false -f k8s/web-deployment.yaml
                kubectl apply --validate=false -f k8s/web-service.yaml
                """
            }
        }

        /* ============================
           5. PROMETHEUS / GRAFANA STAGE
           ============================ */
        stage("Prometheus / Grafana Stage") {
            steps {
                sh """
                helm repo add prometheus-community https://prometheus-community.github.io/helm-charts || true
                helm repo update

                kubectl create namespace monitoring || true

                helm upgrade --install prometheus kube-prometheus-stack \
                  --repo https://prometheus-community.github.io/helm-charts \
                  --namespace monitoring
                """
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully"
        }
        failure {
            echo "CI/CD Pipeline failed"
        }
    }
}
