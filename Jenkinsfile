pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        SSH_GITHUB_CREDENTIALS = credentials('jenkins_ssh_github')
        GITHUB_REPO_CREDENTIALS = credentials('github-repository') // Ensure this contains your PAT
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
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Update Deployment YAML') {
            steps {
                script {
                    // Update deployment.yaml file with the new image version
                    sh """
                        sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${VERSION}|' dev/deployment.yaml
                    """
                    
                    // Configure git and push changes
                    withCredentials([usernamePassword(credentialsId: 'github-repository', usernameVariable: 'yahav123456', passwordVariable: 'jenkins_new_token_github')]) {
                        sh """
                            git config user.name 'yahav123456'
                            git config user.email 'yahavbs100@gmail.com'
                            git add dev/deployment.yaml
                            git commit -m 'Update deployment to ${DOCKER_IMAGE}:${VERSION}'
                            git push https://${GITHUB_USERNAME}:${GITHUB_TOKEN}@github.com/yahav123456/k8s_project.git main
                        """
                    }
                }
            }
        }
    }
}
