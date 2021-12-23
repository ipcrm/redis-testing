 pipeline {

   agent { label 'docker' }

   environment {
     LW_ACCESS_TOKEN = credentials('lw-access-token')
     LW_ACCOUNT_NAME = credentials('lw-account-name')
     K8S_CLUSTER_NAME = credentials('k8s-cluster-name')
     K8S_CLUSTER_CONTEXT = credentials('k8s-context-name')
     K8S_SERVER_URL = credentials('k8s-server-url')
     IMAGE_NAME = "ipcrm/redis"
   }

   stages {
     stage('checkout') {
       steps {
         checkout scm
       }
     }
     stage('build and push') {
       steps {
         script {
           docker.build("${env.IMAGE_NAME}:${env.GIT_COMMIT}")
         }
       }
     }
     stage('Scan') {
       steps {
           sh "curl -L https://github.com/lacework/lacework-vulnerability-scanner/releases/latest/download/lw-scanner-linux-amd64 -o lw-scanner"
           sh "chmod +x lw-scanner"
           sh "./lw-scanner image evaluate ${IMAGE_NAME} ${GIT_COMMIT} --build-id ${BUILD_ID} --build-plan ${JOB_NAME} --save --html --html-file scanner-report.html"
       }
     }
     stage('Push image') {
       steps {
         script {
           docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-access-token') {            
             def newImage = docker.image("${env.IMAGE_NAME}:${env.GIT_COMMIT}")            
             newImage.push()
             newImage.push("latest")
           }
         }
       }    
     }
     stage('k8s') {
       steps{
         withKubeConfig(
           clusterName: env.K8S_CLUSTER_NAME,
           contextName: env.K8S_CLUSTER_CONTEXT,
           credentialsId: 'k8s-build-robot-token',
           namespace: '',
           serverUrl: env.K8S_SERVER_URL) {
             sh "cat ./deploy/redis.yml | TAG=${GIT_COMMIT} envsubst | kubectl apply -f -"
           }
         }
      }
   }
   post {
        always {
            archiveArtifacts artifacts: 'scanner-report.html', fingerprint: true
        }
    }
}

