pipeline {
    agent any

    triggers {
        githubPush() // Triggers on GitHub webhook push
    }

    environment {
        REGISTRY = "docker.io"
        REGISTRY_USER = "abinaya780"
        IMAGE_NAME = "my-app"
        K8S_DEPLOYMENT = "my-app"
        K8S_NAMESPACE = "default"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $REGISTRY_USER/$IMAGE_NAME:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred-id', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin $REGISTRY"
                    sh "docker push $REGISTRY_USER/$IMAGE_NAME:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl set image deployment/$K8S_DEPLOYMENT $IMAGE_NAME=$REGISTRY_USER/$IMAGE_NAME:${BUILD_NUMBER} -n $K8S_NAMESPACE
                    kubectl rollout status deployment/$K8S_DEPLOYMENT -n $K8S_NAMESPACE
                    """
                }
            }
        }
    }
}
