pipeline { 
    agent any 
 
    environment { 
        DOCKER_USER = 'abinaya780' 
        DOCKER_PASS = credentials('dockerhub-creds')  // create this in Jenkins Credentials 
        IMAGE_NAME = 'flask-demo' 
    } 
 
    stages { 
        stage('Clone Repository') { 
            steps { 
                git 'https://github.com/abi780/ci-cd-demo-flask.git' 
            } 
        } 
 
        stage('Build Docker Image') { 
            steps { 
                sh 'docker build -t $DOCKER_USER/$IMAGE_NAME:latest .' 
            } 
        } 
 
        stage('Push to Docker Hub') { 
            steps { 
                sh ''' 
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin 
                    docker push $DOCKER_USER/$IMAGE_NAME:latest 
                ''' 
            } 
        } 
 
        stage('Deploy to Kubernetes') { 
            steps { 
                sh ''' 
                    kubectl apply -f kubernetes/deployment.yaml 
                    kubectl apply -f kubernetes/service.yaml 
                ''' 
            } 
        } 
    } 
} 
