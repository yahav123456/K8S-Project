pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        GIT_CREDENTIALS = 'jenkins-github'
        DOCKER_IMAGE = "yahav12321/k8stest"
        kubeconfigId = "k8sconfigkube"
        VERSION = "${env.BUILD_NUMBER}"
        KUBE_CONFIG = Credentials('k8s_file')
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
            steps {
                script {
                    writeFile(file: 'config', text: "${env.KUBE_CONFIG}")
                    sh "kubectl --kubeconfig=config set image deployment/flask-app flask-app=${DOCKER_IMAGE}:${VERSION} --record"
                }
            }
        }
    }
}
