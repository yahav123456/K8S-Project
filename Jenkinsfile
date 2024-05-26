pipeline {
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: docker
                image: docker:20.10.7-dind
                securityContext:
                  privileged: true
                volumeMounts:
                - name: docker-socket
                  mountPath: /var/run/docker.sock
                - name: jenkins-workspace
                  mountPath: /home/jenkins/agent
              - name: kubectl
                image: bitnami/kubectl:latest
                command:
                - cat
                tty: true
                volumeMounts:
                - name: jenkins-workspace
                  mountPath: /home/jenkins/agent
            volumes:
              - name: docker-socket
                hostPath:
                  path: /var/run/docker.sock
              - name: jenkins-workspace
                emptyDir: {}
            """
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
                container('docker') {
                    script {
                        dockerImage = docker.build("${DOCKER_IMAGE}:${VERSION}")
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
                container('kubectl') {
                    script {
                        sh """
                        kubectl config use-context ${KUBERNETES_CONTEXT}
                        kubectl set image deployment/flask-app flask-app=${DOCKER_IMAGE}:${VERSION} --record
                        """
                    }
                }
            }
        }
    }
}
