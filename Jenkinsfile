pipeline {
    agent {
        kubernetes {
            label 'jenkins-slave1'
            defaultContainer 'jnlp'
        }
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        GIT_CREDENTIALS = 'jenkins-github'
        DOCKER_IMAGE = "yahav12321/k8stest"
        KUBERNETES_CONTEXT = "kind-kind"
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
                container('jnlp') {
                    script {
                        sh '''
                        docker build -t ${DOCKER_IMAGE}:${VERSION} .
                        '''
                    }
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                container('jnlp') {
                    script {
                        sh '''
                        echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                        docker push ${DOCKER_IMAGE}:${VERSION}
                        '''
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                container('jnlp') {
                    script {
                        sh '''
                        kubectl config use-context ${KUBERNETES_CONTEXT}
                        kubectl set image deployment/flask-app flask-app=${DOCKER_IMAGE}:${VERSION} --record
                        '''
                    }
                }
            }
        }
    }
}
