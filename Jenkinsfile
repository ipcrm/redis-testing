 pipeline {

   agent { label 'docker' }

   environment {
     LW_ACCESS_TOKEN = credentials('lw-access-token')
     LW_ACCOUNT_NAME = credentials('lw-account-name')
     IMAGE_NAME = "detcaccounts/redis"
   }

   stages {
     stage('checkout') {
       checkout scm
     }
     stage('build and push') {
       tag = env.BUILD_ID
       def customImage = docker.build("${env.IMAGE_NAME}:${env.BUILD_ID}")
       // customImage.push()
     }
     stage('Scan') {
       steps {
           sh "curl -L https://github.com/lacework/lacework-vulnerability-scanner/releases/latest/download/lw-scanner-linux-amd64 -o lw-scanner"
           sh "chmod +x lw-scanner"
           sh "./lw-scanner image evaluate ${IMAGE_NAME} ${tag} --build-id ${BUILD_ID} --build-plan ${JOB_NAME} --save"
       }
     }
     stage('k8s') {
       steps{
         withKubeConfig(
           clusterName: credentials("k8s-cluster-name"),
           contextName: credentials("k8-context-name"),
           credentialsId: 'k8s-build-robot-token',
           namespace: '',
           serverUrl: credentials("k8s-server-url")) {
             sh 'kubectl get pods'
           }
         }
      }
   }
}

