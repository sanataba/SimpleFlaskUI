pipeline {
      agent {
        label 'new123'
    }
 /*   environment {
        registry = "sanataba/python" 
    } */
      environment {
        registry = "706274417810.dkr.ecr.ap-south-1.amazonaws.com/python"
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
               /*     img = registry + ":${env.BUILD_ID}"
                    println ("${img}")
                    dockerImage = docker.build("${img}") */
                   sh 'docker build -t python .'
                   sh 'docker tag python:latest 706274417810.dkr.ecr.ap-south-1.amazonaws.com/python:latest'
                    
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

                sh label: '', script: "docker run -d --name sana -p 5000:5000 python:latest"
          }
        } 
   /*     stage('push docker image'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'dockerhubpwd', variable: 'dockerhubpwd')]) {
                    sh 'docker login -u sanataba -p ${dockerhubpwd}'
                  //  sh 'docker push sanataba/python'
                    dockerImage.push()
}
                    
                }
            }
        } */
        
        stage('aws login'){
            steps{
              sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 706274417810.dkr.ecr.ap-south-1.amazonaws.com'
          }
      } 
             stage('DockerPush to AWS ECR'){
            steps{
                sh 'docker push 706274417810.dkr.ecr.ap-south-1.amazonaws.com/python:latest'
            }
        } 
        
        stage ('eks Deploy') {
           steps {
               script{
                    withKubeConfig([credentialsId: 'K8s']) 
                    {
        
              sh 'kubectl apply -f Deployment.yaml'
        }
        }
    
      }
}
}
}
