
pipeline {
    agent any

    triggers {
        githubPush() // Triggers on GitHub webhook push
    }

    environment {
        IMAGE_NAME = "flask-demo"
        K8S_NAMESPACE = "default"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/abi780/ci-cd-demo-flask.git'
            }
        }

        stage('Set Build Tag') {
            steps {
                script {
                    // Short commit hash for tagging
                    COMMIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    echo "Build Tag: ${COMMIT_HASH}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:latest -t ${IMAGE_NAME}:${COMMIT_HASH} .
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag ${IMAGE_NAME}:latest $DOCKER_USER/${IMAGE_NAME}:latest
                        docker tag ${IMAGE_NAME}:${COMMIT_HASH} $DOCKER_USER/${IMAGE_NAME}:${COMMIT_HASH}
                        docker push $DOCKER_USER/${IMAGE_NAME}:latest
                        docker push $DOCKER_USER/${IMAGE_NAME}:${COMMIT_HASH}
                    """
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                script {
                    sh """
                        sed -i 's|image: .*|image: ${DOCKER_USER}/${IMAGE_NAME}:${COMMIT_HASH}|' k8s/deployment.yaml
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}
                    kubectl rollout status deployment/${IMAGE_NAME} -n ${K8S_NAMESPACE}
                """
            }
        }
    }
}
