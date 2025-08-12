pipeline { 
    agent any 
 
    environment { 
        IMAGE_NAME = 'flask-demo'
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
                    COMMIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    echo "Using commit hash: ${COMMIT_HASH}"
                }
            }
        }
 
        stage('Build Docker Image') { 
            steps { 
                sh '''
                    docker build -t ${IMAGE_NAME}:latest -t ${IMAGE_NAME}:${COMMIT_HASH} .
                '''
            } 
        } 
 
        stage('Push to Docker Hub') { 
            steps { 
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag ${IMAGE_NAME}:latest $DOCKER_USER/${IMAGE_NAME}:latest
                        docker tag ${IMAGE_NAME}:${COMMIT_HASH} $DOCKER_USER/${IMAGE_NAME}:${COMMIT_HASH}
                        docker push $DOCKER_USER/${IMAGE_NAME}:latest
                        docker push $DOCKER_USER/${IMAGE_NAME}:${COMMIT_HASH}
                    '''
                }
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
