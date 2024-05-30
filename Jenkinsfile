pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        SSH_GITHUB_CREDENTIALS = 'jenkins_ssh_github'
        DOCKER_IMAGE = "yahav12321/k8stest"
        VERSION = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yahav123456/k8s_project.git', credentialsId: "${SSH_GITHUB_CREDENTIALS}"
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

        stage('Update Deployment YAML') {
            steps {
                script {
                    // עדכון קובץ ה-deployment.yaml עם גרסת ה-Image החדשה
                    sh """
                        sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${VERSION}|' dev/deployment.yaml
                    """
                    
                    // ביצוע commit ו-push ל-GitHub
                    sh """
                        git config user.name 'yahav123456'
                        git config user.email 'yahavbs100@gmail.com'
                        git add dev/deployment.yaml
                        git commit -m 'Update deployment to ${DOCKER_IMAGE}:${VERSION}'
                        git push yahav123456/k8s_project main
                    """
                }
            }
        }
    }
}
