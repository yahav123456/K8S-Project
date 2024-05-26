pipeline {
    agent {
        kubernetes {
            label 'jenkins-slave'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: 'gcr.io/kaniko-project/executor:latest'
    command:
    - /busybox/cat
    tty: true
"""
        }
    }
    stages {
        stage('Install Kaniko') {
            steps {
                container('jnlp') {
                    sh '''
                    curl -LO https://storage.googleapis.com/kaniko-release/release/v1.6.0/kaniko-linux-amd64
                    chmod +x kaniko-linux-amd64
                    mv kaniko-linux-amd64 /kaniko/executor
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('kaniko') {
                    sh '/kaniko/executor --dockerfile=Dockerfile --context=. --destination=yahav12321/k8stest:20'
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    container('kaniko') {
                        sh "echo \${DOCKERHUB_PASSWORD} | docker login --username \${DOCKERHUB_USERNAME} --password-stdin"
                        sh "docker push yahav12321/k8stest:20"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                container('jnlp') {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}
