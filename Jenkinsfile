pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        GIT_CREDENTIALS = 'jenkins-github'
        DOCKER_IMAGE = "yahav12321/k8stest"
        VERSION = "${env.BUILD_NUMBER}"
        ARGOCD_API_TOKEN = credentials('argocd-api-token')
        ARGOCD_SERVER_URL = "https://localhost:9090" // החלף עם כתובת ה-URL של ה-Argo CD שלך
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yahav123456/k8s_project.git', credentialsId: "${GIT_CREDENTIALS}"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${VERSION}")
                }
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        dockerImage.push()
                    }
                }
            }
        } 

        stage('Sync Argo CD') {
            steps {
                script {
                    def response = sh(script: "curl -X POST -H 'Authorization: Bearer ${ARGOCD_API_TOKEN}' -H 'Content-Type: application/json' -d '{\"revision\": \"HEAD\"}' ${ARGOCD_SERVER_URL}/api/v1/applications/my-app/sync", returnStdout: true)
                    echo response
                }
            }
        }
    }
}
