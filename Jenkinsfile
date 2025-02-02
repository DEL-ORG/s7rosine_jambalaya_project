pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_cred')
        IMAGE_TAG = "V1.0.${env.BUILD_NUMBER}" // Corrected reference to env variable
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
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t rosinebelle/s7rosine_jambalaya:${env.IMAGE_TAG} .
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
                docker push rosinebelle/s7rosine_jambalaya:${env.IMAGE_TAG}
                """
            }
        }

        // stage('Testing') {
        //     agent {
        //         docker { image 'maven:3.8.4-eclipse-temurin-17-alpine' }
        //     }
        //     steps {
        //         sh '''
        //         cd s7rosine_jambalaya
        //         mvn test
        //         '''
        //     }
        // }


        // stage('SonarQube Analysis') {
        //     agent {
        //         docker { image 'sonarsource/sonar-scanner-cli:5.0.1' }
        //     }
        //     environment {
        //         CI = 'true'
        //         scannerHome = '/opt/sonar-scanner'
        //     }
        //     steps {
        //         withSonarQubeEnv('sonar') {
        //             sh "${scannerHome}/bin/sonar-scanner"
        //         }
        //     }
        // }

        stage('Update Helm Repo for ArgoCD') {
            steps {
                sh """
                rm -rf s7rosine_jambalaya_project || true
                git clone -b main git@github.com:DEL-ORG/s7rosine_jambalaya_project.git
                cd s7rosine_jambalaya_project
                sed -i 's/tag:.*/tag: ${env.IMAGE_TAG}/' ./chart/values.yaml
                git config user.email "rosinemuku@yahoo.com"
                git config user.name "rosinebelle"
                git add ./chart/values.yaml
                git commit -m "Update image tag to ${env.IMAGE_TAG}"
                git push origin main
                """
            }
        }
    } // Properly closed the 'stages' block

    post {
        success {
            echo "Docker image built and pushed successfully. ArgoCD will handle deployment."
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
