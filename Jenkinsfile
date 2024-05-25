// Jenkinsfile

pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
        DOCKER_IMAGE = 'yahav12321/k8stest:02'
        KUBERNETES_DEPLOYMENT = 'flask-app'
    }

    triggers {
        githubPush()
    }
    
    stages {
        stage('Build') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        sh 'docker push $DOCKER_IMAGE'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    kubectl set image deployment/$KUBERNETES_DEPLOYMENT $KUBERNETES_DEPLOYMENT=$DOCKER_IMAGE --record
                    kubectl rollout status deployment/$KUBERNETES_DEPLOYMENT
                    '''
                }
            }
        }
    }
}
