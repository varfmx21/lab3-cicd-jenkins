pipeline {
    agent any

    tools {
        nodejs 'NodeJS 7.8.0'
    }

    environment {
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        APP_PORT   = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --watchAll=false'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:v1.0 ."
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Detener y eliminar contenedores previos (mínimo downtime)
                    sh """
                        docker ps -q --filter 'name=${IMAGE_NAME}' | xargs -r docker stop
                        docker ps -aq --filter 'name=${IMAGE_NAME}' | xargs -r docker rm
                    """
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker run -d --name ${IMAGE_NAME} --expose 3000 -p 3000:3000 ${IMAGE_NAME}:v1.0"
                    } else {
                        sh "docker run -d --name ${IMAGE_NAME} --expose 3001 -p 3001:3000 ${IMAGE_NAME}:v1.0"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished for branch: ${env.BRANCH_NAME} on port ${env.APP_PORT}"
        }
    }
}