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
                script {
                    def authorName = sh(
                        script: "git log -1 --pretty=format:'%an'",
                        returnStdout: true
                    ).trim()
                    if (authorName == "jenkins") {
                        currentBuild.result = 'SUCCESS'
                        error "Skipping build due to Jenkins commit"
                    }
                }
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
                    
                    withCredentials([usernamePassword(credentialsId: 'github-repository', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                    sh """
                        git config user.name 'jenkins'
                        git config user.email 'jenkins@example.com'
                        git add dev/deployment.yaml
                        git commit -m 'Update deployment to ${DOCKER_IMAGE}:${VERSION}'
                        git push https://${USERNAME}:${PASSWORD}@github.com/yahav123456/k8s_project.git main
                    """
                    }
                }
            }
        }
    }
}
