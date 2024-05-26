pipeline {
agent {
    kubernetes {
        yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: docker
                image: docker:dind
                env:
                - name: DOCKER_HOST
                  value: tcp://localhost:2375
                securityContext:
                  privileged: true
                volumeMounts:
                - name: docker-sock
                  mountPath: /var/run/docker.sock
              volumes:
              - name: docker-sock
                emptyDir: {}
        '''
    }
}

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        GIT_CREDENTIALS = credentials('jenkins-github')
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
                container('docker') {
                    script {
                        dockerImage = docker.build("${DOCKER_IMAGE}:${VERSION}", '.')
                    }
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                container('docker') {
                    script {
                        docker.withRegistry('', 'dockerhub-credentials') {
                            dockerImage.push()
                        }
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                container('docker') {
                    script {
                        sh """
                        kubectl set image deployment/flask-app kind-control-plane=${DOCKER_IMAGE}:${VERSION} --record
                        """
                    }
                }
            }
        }
    }
}
