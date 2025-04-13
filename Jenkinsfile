pipeline {
    agent any
    environment {
        IMAGE_NAME = "tarunchadaram/maindockimg"
        TAG = "${env.BUILD_ID}"
    }
    stages {
        stage("Checkout Code") {
            steps {
                script {
                    checkout scm
                }
            }
        }
        stage("Build and Push Docker Image") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "echo \$DOCKERHUB_PASSWORD | docker login -u \$DOCKERHUB_USERNAME --password-stdin"
                    }
                    sh "docker build -t $IMAGE_NAME:$TAG ."
                    sh "docker push $IMAGE_NAME:$TAG"
                    sh "docker tag $IMAGE_NAME:$TAG $IMAGE_NAME:latest"
                    sh "docker push $IMAGE_NAME:latest"
                }
            }
        }
        stage("Deploy to Rancher Kubernetes Cluster") {
            steps {
                script {
                    sh "sed -i 's|IMAGE_NAME|$IMAGE_NAME:$TAG|g' deployment.yaml"
                    withCredentials([file(credentialsId: 'kubernetes', variable: 'KUBECONFIG')]) {
                        sh "kubectl --kubeconfig=$KUBECONFIG apply -f deployment.yaml"
                        sh "kubectl --kubeconfig=$KUBECONFIG apply -f service.yaml"
                    }
                }
            }
        }
    }
}
