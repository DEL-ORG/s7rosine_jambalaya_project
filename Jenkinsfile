pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_cred')
        IMAGE_TAG = "V1.0.${BUILD_NUMBER}" // Set the image tag dynamically
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
        timeout(time: 15, unit: 'MINUTES')
        timestamps()
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm // Fetch the code from the repository
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t rosinebelle/s7rosine_jambalaya:${IMAGE_TAG} .
                """
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                docker push rosinebelle/s7rosine_jambalaya:${IMAGE_TAG}
                """
            }
        }

        // Uncomment the following stage for Helm repo update if needed
        stage('Update Helm Repo for ArgoCD') {
            steps {
                sh """
                rm -rf s7rosine_jambalaya || true
                git clone -b main git@github.com:DEL-ORG/s7rosine_jambalaya_project.git
                cd ${WORKSPACE}/springboot/s7rosine_jambalaya_project
                sed -i 's/tag:.*/tag: ${IMAGE_TAG}/' ./chart/values.yaml
                git config user.email "rosinemuku@yahoo.com"
                git config user.name "rosinebelle"
                git add ./chart/values.yaml
                git commit -m "Update image tag to ${IMAGE_TAG}"
                git push origin prod
                """
            }
        }

    } // <-- Properly closing the 'stages' block here

    post {
        success {
            echo "Docker image built and pushed successfully. ArgoCD will handle deployment."
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
