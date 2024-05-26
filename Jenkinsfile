pipeline {
   agent {
       kubernetes {
           yaml """
           apiVersion: v1
           kind: Pod
           spec:
             containers:
             - name: maven
               image: maven:3.8.1-openjdk-8
               command:
               - sleep
               args:
               - 99d
           """
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
                   sh """
                   kubectl set image deployment/flask-app kind-control-plane=${DOCKER_IMAGE}:${VERSION} --record
                   """
               }
           }
       }
   }
}
