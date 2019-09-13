\pipeline {
 agent any
 stages{
      stage('checkout')
      {
          steps{
              withCredentials([usernameColonPassword(credentialsId: 'gitCredentials', variable: 'gitCredentilas')]) {
                  git 'https://github.com/sacpv/devops-case-study'
              }
          }
      }
      
     stage('Build & Run Junit'){
         steps{
        withMaven(maven:'MyMaven'){
          dir('DevOpsCaseStudy'){
            sh 'chmod +x src/main/resources/chromedriver'
            sh '''
            mvn clean install -B -U -q -Dmaven.test.failure.ignore=true -DskipTests
            '''
            junit 'target/surefire-reports/**/*.xml'
          }
         }
        }
      }
  
      stage('compile'){
          steps{
           withMaven(maven:'MyMaven'){
           sh 'mvn compile'
           }
          }
      }
      stage('SonarQube analysis') {
      steps {
          withMaven(maven:'MyMaven'){
          sh 'mvn sonar:sonar -Dsonar.host.url=http://34.93.134.185:9000'
          }
      }
      }
      
     stage('Build & Push Image'){
         steps{
          dir('ci_helper'){
            sh 'cp /var/lib/jenkins/workspace/CaseStudy-Pipeline/DevOpsCaseStudy/target/DevOpsCaseStudy.war .'
            sh "sudo docker build -t sachinvarghese333/app:${env.BUILD_NUMBER} ."
            withCredentials([usernamePassword(credentialsId: 'dockerCredentials', passwordVariable: 'docker_password', usernameVariable: 'docker_username')]) {
              sh "sudo docker login -u=${docker_username} -p=${docker_password}"
              sh 'sudo docker push sachinvarghese333/app'
            }
           }
          }
        }
        
       stage('Deploy-Docker'){
          steps{
          sh "sudo docker pull sachinvarghese333/app:${env.BUILD_NUMBER}"
          sh "sudo docker run -itd --name app${env.BUILD_NUMBER} --rm -p 8081:8080 sachinvarghese333/app:${env.BUILD_NUMBER}"          
          sh 'sudo docker logout'
          }
        }
        
        stage('Run Selenium Tests'){
        steps{
          dir('DevOpsCaseStudy'){
            withMaven(maven:'MyMaven'){
            sh 'mvn failsafe:integration-test -Dskip.surefire.tests -Dapp.url=http://localhost:8081/DevOpsCaseStudy/'
            junit 'target/failsafe-reports/**/*.xml'
            sh "sudo docker stop app${env.BUILD_NUMBER}"
            }
          }
         } 
        }
        
       stage('Prod-Deployment'){
         steps{
          dir('ci_helper/kubernetes/deploy'){
              
              sh  'script : deploy.sh'
            }
           }
         }
     
  }
           post {  
         always {  
             echo 'This will always run'  
         }  
         success {  
             mail bcc: '', body: "<b>Example</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "ERROR CI: Project name -> ${env.JOB_NAME}", to: "thomas.mjames95@gmail.com";  
         }          
         failure {  
             mail bcc: '', body: "<b>Example</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "ERROR CI: Project name -> ${env.JOB_NAME}", to: "thomas.mjames95@gmail.com";  
         }  
         unstable {  
             echo 'This will run only if the run was marked as unstable'  
         }  
         changed {  
             echo 'This will run only if the state of the Pipeline has changed'  
             echo 'For example, if the Pipeline was previously failing but is now successful'  
         }  
     } 
}
