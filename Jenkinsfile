pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "sujisuki/backend-app:latest"    // Change this to your Docker Hub repo
        FRONTEND_IMAGE = "sujisuki/frontend-app:latest"  // Change this to your Docker Hub repo
        BACKEND_CONTAINER = "backend-running-app"
        FRONTEND_CONTAINER = "frontend-running-app"
        REGISTRY_CREDENTIALS = "docker_suji"  // Jenkins credentials ID for Docker Hub
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github_suji', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/SujithaKC/Jenkins_E-commerce.git", branch: 'main'
                }
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh 'docker build -t $BACKEND_IMAGE .'
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh 'docker build -t $FRONTEND_IMAGE .'
                }
            }
        }

        stage('Login to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_suji', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                parallel {
                    stage('Push Backend Image') {
                        steps {
                            sh 'docker push $BACKEND_IMAGE'
                        }
                    }

                    stage('Push Frontend Image') {
                        steps {
                            sh 'docker push $FRONTEND_IMAGE'
                        }
                    }
                }
            }
        }

        stage('Stop & Remove Existing Containers') {
            steps {
                script {
                    sh '''
                    docker stop $BACKEND_CONTAINER $FRONTEND_CONTAINER || true
                    docker rm $BACKEND_CONTAINER $FRONTEND_CONTAINER || true
                    '''
                }
            }
        }

        stage('Run Backend Container') {
            steps {
                sh 'docker run -d -p 5000:5000 --name $BACKEND_CONTAINER $BACKEND_IMAGE'
            }
        }

        stage('Run Frontend Container') {
            steps {
                sh 'docker run -d -p 3000:3000 --name $FRONTEND_CONTAINER $FRONTEND_IMAGE'
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Backend and Frontend are running."
        }
        failure {
            echo "❌ Deployment failed! Check logs for errors."
        }
    }
}
