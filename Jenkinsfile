pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'sathish1918/python'
        DOCKER_TAG = "1.0.${env.BUILD_NUMBER}"
        STAGING_SERVER = 'http://52.66.36.217:8080/'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sathishkumar10102001/githubemc1'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    // Example for Node.js app - adjust for your tech stack
                    docker.image('node:14').inside {
                        sh 'npm install'
                        sh 'npm test'
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://hub.docker.com/u/sathish1918', 'docker-hub-credentials') {
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push()
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                script {
                    sshagent(['staging-server-credentials']) {
                        sh """
                            ssh user@${env.STAGING_SERVER} "
                                docker pull ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
                                docker stop python || true
                                docker rm python || true
                                docker run -d --name python -p 3000:3000 ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
                            "
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            slackSend(color: 'good', message: "Build ${env.BUILD_NUMBER} succeeded!")
        }
        failure {
            slackSend(color: 'danger', message: "Build ${env.BUILD_NUMBER} failed!")
        }
    }
}
