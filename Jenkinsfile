pipeline {
    agent {
        label 'slave-node' // Use the label of the Jenkins slave agent
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-cred')
        IMAGE_TAG = "V1.0.${BUILD_NUMBER}" // Set the image tag dynamically
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
        timeout(time: 60, unit: 'MINUTES')
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
                docker build -t rosinebelle/springboot/s7rosine_jambalaye:${IMAGE_TAG} .
                """
            }
        }

        stage('Login to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                docker push rosinebelle/springboot/s7rosine_jambalaya:${IMAGE_TAG}
                """
            }
        }

    //     stage('Update Helm Repo for ArgoCD') {
    //         steps {
    //             sh """
    //             rm -rf s7rosine_jambalaya || true
    //             git clone -b prod git@github.com:DEL-ORG/s7rosine_jambalaya_project.git
    //             cd ${WORKSPACE}/springboot/s7rosine_jambalaya
    //             sed -i 's/tag:.*/tag: ${IMAGE_TAG}/' ./chart/values.yaml
    //             git config user.email "rosinemuku@yahoo.com"
    //             git config user.name "rosinebelle"
    //             git add ./chart/values.yaml
    //             git commit -m "Update image tag to ${IMAGE_TAG}"
    //             git push origin prod
    //             """
    //         }
    //     }
    // }
    post {
        success {
            echo "Docker image built and pushed successfully. ArgoCD will handle deployment."
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
