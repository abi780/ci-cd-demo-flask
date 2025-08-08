pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/abi780/ci-cd-demo-flask.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip3 install --user -r requirements.txt'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-demo:latest .'
            }
        }

       stage('Push to Docker Hub') {
           steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh """
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                docker tag flask-demo:latest $DOCKER_USER/flask-demo:latest
                docker push $DOCKER_USER/flask-demo:latest
            """
        }
    }
}


        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
}
