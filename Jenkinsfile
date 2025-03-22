pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "yugeshwarangm/backend-app1:latest"
        FRONTEND_IMAGE = "yugeshwarangm/frontend-app1:latest"
        BACKEND_CONTAINER = "backend-running-app-day5"
        FRONTEND_CONTAINER = "frontend-running-app-day5"
        REGISTRY_CREDENTIALS = "docker-yugesh"
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-yugeshwaran', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/Yugeshwaran-gm/terraform.git", branch: 'main'
                }
            }
        }

        stage('Build & Push Backend Image') {
            steps {
                dir('backend') {  // ✅ Ensure the backend directory exists
                    sh "docker build -t ${BACKEND_IMAGE} ."
                    withCredentials([usernamePassword(credentialsId: 'docker-yugesh', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${BACKEND_IMAGE}"
                    }
                }
            }
        }

        stage('Build & Push Frontend Image') {
            steps {
                dir('frontend') {  // ✅ Ensure the frontend directory exists
                    sh "docker build -t ${FRONTEND_IMAGE} ."
                    withCredentials([usernamePassword(credentialsId: 'docker-yugesh', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${FRONTEND_IMAGE}"
                    }
                }
            }
        }

        stage('Stop & Remove Existing Containers') {
            steps {
                script {
                    sh '''
                    if [ "$(docker ps -aq -f name=${BACKEND_CONTAINER})" ]; then
                        docker stop ${BACKEND_CONTAINER} || true
                        docker rm ${BACKEND_CONTAINER} || true
                    fi

                    if [ "$(docker ps -aq -f name=${FRONTEND_CONTAINER})" ]; then
                        docker stop ${FRONTEND_CONTAINER} || true
                        docker rm ${FRONTEND_CONTAINER} || true
                    fi
                    '''
                }
            }
        }

        stage('Run Backend Container') {
            steps {
                sh "docker run -d -p 5000:5000 --name ${BACKEND_CONTAINER} ${BACKEND_IMAGE}"  // ✅ Matched friend's port
            }
        }

        stage('Run Frontend Container') {
            steps {
                sh "docker run -d -p 3000:3000 --name ${FRONTEND_CONTAINER} ${FRONTEND_IMAGE}"  // ✅ Matched friend's port
            }
        }
    }

	stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    echo "Applying Kubernetes configurations..."
                    
                    # Replace the image in the deployment files (optional)
                    sed -i "s|IMAGE_PLACEHOLDER_BACKEND|${BACKEND_IMAGE}|" k8s/backend-deployment.yaml
                    sed -i "s|IMAGE_PLACEHOLDER_FRONTEND|${FRONTEND_IMAGE}|" k8s/frontend-deployment.yaml

                    # Apply Kubernetes resources
                    kubectl apply -f k8s/backend-deployment.yaml
                    kubectl apply -f k8s/frontend-deployment.yaml
                    kubectl apply -f k8s/backend-service.yaml
                    kubectl apply -f k8s/frontend-service.yaml

                    echo "Deployment completed!"
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh '''
                    echo "Checking Kubernetes pod status..."
                    kubectl get pods -o wide
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Build, push, and container execution successful!"
        }
        failure {
            echo "Build or container execution failed."
        }
    }
}
