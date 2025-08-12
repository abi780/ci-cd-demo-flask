pipeline {
    agent any

    environment {
        REGISTRY       = "docker.io/your_dockerhub_username"  // Change to your registry
        IMAGE_NAME     = "ci-cd-demo-flask"
        KUBECONFIG_CRED = "kubeconfig-credentials-id"         // Jenkins Kubeconfig credential ID
        DOCKER_CRED    = "dockerhub-credentials-id"           // Jenkins Docker Hub credential ID
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/abi780/ci-cd-demo-flask.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def tag = "v${env.BUILD_NUMBER}"
                    sh """
                        docker build -t ${REGISTRY}/${IMAGE_NAME}:${tag} .
                        docker tag ${REGISTRY}/${IMAGE_NAME}:${tag} ${REGISTRY}/${IMAGE_NAME}:latest
                    """
                    env.IMAGE_TAG = tag
                }
            }
        }

        stage('Push to Docker Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                            docker push ${REGISTRY}/${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG_FILE')]) {
                        sh """
                            export KUBECONFIG=$KUBECONFIG_FILE
                            sed -i 's|IMAGE_PLACEHOLDER|${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml
                            kubectl apply -f k8s/deployment.yaml
                        """
                    }
                }
            }
        }
    }
}
