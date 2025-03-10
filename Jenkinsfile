pipeline {
    agent any
    stages {
        stage("Checkout and Build a Docker image for the web app") {
            steps {
                script {
                    checkout scm
                    withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                    }
                    sh "docker build -t tarunchadaram/maindockimg:${env.BUILD_ID} ."
                }
            }
        }
        stage("Pushing image to DockerHub") {
            steps {
                script {
                    sh "docker push tarunchadaram/maindockimg:${env.BUILD_ID}"
                }
            }
        }
        stage("Deploy to Rancher Kubernetes Cluster") {
            steps {
                script {
                    sh "sed -i 's|IMAGE_NAME|tarunchadaram/maindockimg:${env.BUILD_ID}|g' deployment.yaml"
                    withCredentials([file(credentialsId: 'kubernetes', variable: 'KUBECONFIG')]) {
                        sh "kubectl --kubeconfig=$KUBECONFIG apply -f deployment.yaml"
                        sh "kubectl --kubeconfig=$KUBECONFIG apply -f service.yaml"
                    }
                }
            }
        }
    }
}
