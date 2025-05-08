pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'sathish1918/python'
        DOCKER_TAG = "1.0.${env.BUILD_NUMBER}"
        STAGING_SERVER = '52.66.36.217'  // Removed http:// and port from here
        STAGING_PORT = '8080'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/sathishkumar10102001/githubemc1.git',
                credentialsId: 'github-credentials'  // Added credentials
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    // Changed to Python testing since your image is python-based
                    docker.image('python:3.9').inside {
                        sh 'pip install -r requirements.txt'
                        sh 'python -m pytest'  // Assuming pytest is used
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')  // Optional: also push as latest
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                script {
                    sshagent(['staging-server-credentials']) {
                        sh """
                            ssh -p ${STAGING_PORT} ubuntu@${STAGING_SERVER} "
                                docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                                docker stop python || true
                                docker rm python || true
                                docker run -d --name python -p 3000:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                            "
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()  // Clean workspace after build
        }
        success {
            slackSend(
                color: 'good', 
                message: "SUCCESS: Build ${BUILD_NUMBER} deployed to staging\nImage: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "FAILED: Build ${BUILD_NUMBER}\nCheck console: ${BUILD_URL}"
            )
        }
    }
}
