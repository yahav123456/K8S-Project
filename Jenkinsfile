pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        GIT_CREDENTIALS = 'jenkins-github'
        DOCKER_IMAGE = "yahav12321/k8stest"
        VERSION = "${env.BUILD_NUMBER}"
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
        
        stage('Deploy to Kubernetes') {
            environment {
                KUBECONFIG = credentials('k88s')
            }
            steps {
                script {
                    def deploymentName = "flask-app"
                    def containerName = "flask-app"
                    def image = "${DOCKER_IMAGE}:${VERSION}"
                    def namespace = "deafult"

                    sh "kubectl set image deployment/${deploymentName} ${containerName}=${image} -n ${namespace} --record"
                }
            }
        }
    }
}
