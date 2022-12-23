pipeline {
    agent {
        label 'sana_123'
    }
    environment {
        registry = "sanataba/python" 
    }
      stages{
        stage('git checkout'){
           steps{
               checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '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', url: 'https://github.com/sanataba/SimpleFlaskUI']]])
           }
        }
        stage('docker build'){
            steps{
                script{
                    img = registry + ":${env.BUILD_ID}"
                    println ("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }
        
        stage ('Stop previous running container'){
            steps{
                sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${JOB_NAME} | awk \'{print $1}\')'
            //    sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force' //this will delete all images
                sh returnStatus: true, script: 'docker rm ${JOB_NAME}'
            }
        }
        
        stage('Test - Run Docker Container on Jenkins node') {
           steps {

                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 5000:5000 ${img}"
          }
        }
        stage('push docker image'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'dockerhubpwd', variable: 'dockerhubpwd')]) {
                    sh 'docker login -u sanataba -p ${dockerhubpwd}'
                  //  sh 'docker push sanataba/python'
                    dockerImage.push()
                       }
                    
                }
            }
        }
          
        stage ('K8S Deploy') {
           steps {
              script {
                 kubernetesDeploy(
                    configs: 'k8s-deployment.yaml',
                    kubeconfigId: 'K8S',
                    enableConfigSubstitution: true
                    )           
               
            }
        }
    }
      }
}
