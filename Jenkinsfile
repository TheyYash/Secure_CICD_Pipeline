//pipeline code
pipeline {
    agent any
    stages {
        stage('Networking Configuration') {
            steps {
                sh 'docker network ls'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'cd $WORKSPACE'
                sh 'rm -rf project'
                git branch: "master",
                    url: "https://github.com/TheyYash/Secure_CICD_Pipeline.git"
                sh 'ls'
            }
        }
        stage('Pre-Build Tests') {
            parallel {
                stage('Git Repository Scanner') {
                    steps {
                        sh 'cd $WORKSPACE'
                        sh 'trufflehog https://github.com/RaziAbbas1/Devsecops --json | jq "{branch:.branch, commitHash:.commitHash, path:.path, stringsFound:.stringsFound}" > trufflehog_report.json || true'
                        sh 'cat trufflehog_report.json'
                        sh 'echo "Scanning Repositories.....done"'
                        archiveArtifacts artifacts: 'trufflehog_report.json', onlyIfSuccessful: true                      
                        emailext attachLog: true, attachmentsPattern: 'trufflehog_report.json',
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Thankyou,\n CDAC-Project Group-7", 
                        subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME} - success", mimeType: 'text/html', to: "daddu1017@gmail.com"
                    }
                }
                stage('Image Security') {
                    steps {
                        sh 'cd $WORKSPACE'
                        sh 'dockle --input ~/docker_img_backup/mytomcat.tar -f json -o mytomcat_report.json'
                        sh 'cat mytomcat_report.json | jq {summary}'
                        sh 'dockle --input ~/docker_img_backup/pgadmin4.tar -f json -o pgadmin4_report.json'
                        sh 'cat pgadmin4_report.json | jq {summary}'
                        sh 'dockle --input ~/docker_img_backup/postgres11.tar -f json -o postgres11_report.json'
                        sh 'cat postgres11_report.json | jq {summary}'
                        sh 'dockle --input ~/docker_img_backup/zap2docker.tar -f json -o zap2docker-stable_report.json'
                        sh 'cat zap2docker-stable_report.json | jq {summary}'
                        sh 'dockle --input ~/docker_img_backup/sonarqube.tar -f json -o sonarqube_report.json'
                        sh 'cat sonarqube_report.json | jq {summary}'
                        archiveArtifacts artifacts: '*.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*.json', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "daddu1017@gmail.com"
                     }
                  }
              }
        }
    //    stage('Build Stage') {
  //          steps {
  //              sh 'mvn clean'
    //            sh 'mvn compile'
  //             sh 'mvn package'
  //          }
  //      }
       
        stage('SCA') {
            parallel {
                stage('Dependency Check') {
                    steps {
                        sh 'wget https://github.com/TheyYash/Secure_CICD_Pipeline/blob/master/dc.sh'
                        sh 'chmod +x dc.sh'
                        sh './dc.sh'
                        //sh 'chmod +x odc-reports'
                        archiveArtifacts artifacts: 'odc-reports/*.html', onlyIfSuccessful: true
                        archiveArtifacts artifacts: 'odc-reports/*.csv', onlyIfSuccessful: true
                        archiveArtifacts artifacts: 'odc-reports/*.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*.html', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "daddu1017@gmail.com"
                    }
                }
                stage('Junit Testing') {
                    steps {
                        sh 'echo "Junit Reports are created using archiveArtifacts"'
                        archiveArtifacts artifacts: '*junit.xml', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*junit.xml', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-11",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "daddu1017@gmail.com"
                    }
                }
            }
        }
           stage('SonarQube Analysis') {
             steps {
                 // sh 'docker container stop sonarqube || true'
                 // sh 'docker container rm -f sonarqube || true'
                 // sh 'docker run -p 9000:9000 -d --name sonarqube owasp/sonarqube'
                // sshagent(['dockerserver']) {
                 withSonarQubeEnv('sonar') {
                     sh 'mvn sonar:sonar'
                 //    sh 'cat /var/lib/jenkins/workspace/sonarqube_report.txt'
                // }                      
         }
      }
    }
      //  stage('SonarQube Analysis report') {
        //    steps {
        //            sh 'mvn sonar:sonar Dsonar.projectKey=sonarqube -Dsonar.host.url=http://192.168.80.128:9000 -Dsonar.login=99eff6549cfb87f043869711c08471036fbbcc21'
       //     }
     //   }
         stage('Build Stage') {
            steps {
                sh 'mvn clean'
                sh 'mvn compile'
                sh 'mvn package'
            }
        }
               stage('Build Docker Images') {
                     steps {
                          sh 'docker build -t $JOB_NAME:v1.$BUILD_ID .'
                          sh 'docker image tag $JOB_NAME:v1.$BUILD_ID nani123456789/$JOB_NAME:v1.$BUILD_ID'
                          sh 'docker image tag $JOB_NAME:v1.$BUILD_ID nani123456789/$JOB_NAME:latest'
                  
              }
        }
       stage('Push Image To Docker Hub') { 
             steps {
                           withCredentials([string(credentialsId: 'docker', variable: 'docker')]) {
                           sh 'docker login -u nani123456789 -p ${docker}'
               }
                           sh 'docker image push nani123456789/$JOB_NAME:v1.$BUILD_ID'
                           sh 'docker image push nani123456789/$JOB_NAME:latest'
                           sh 'docker rmi $JOB_NAME:v1.$BUILD_ID nani123456789/$JOB_NAME:v1.$BUILD_ID nani123456789/$JOB_NAME:latest'
                   }
               }    
            stage('Deploying Containers') {
                  steps {  
                        script {
                        //   sh 'ssh -o StrictHostKeyChecking=no yash@192.168.80.140 sudo docker pull nani123456789/pipeline'
                        //   sh 'ssh -o StrictHostKeyChecking=no yash@192.168.80.140 sudo docker run -p 8000:8000 -d nani123456789/$JOB_NAME:latest'
                        //   def dockerrun = 'sudo docker run -p 8080:8080 -d nani123456789/$JOB_NAME:latest'
                        //   def dockerrm = 'docker container rm -f Devsecops'
                        //   def dockerimg = 'docker rmi nani123456789/$JOB_NAME'
                           sshagent(['dockerserver']) {
                           sh 'ssh -o StrictHostKeyChecking=no yash@192.168.80.140 sudo docker pull nani123456789/pipeline'
                           sh 'ssh -o StrictHostKeyChecking=no yash@192.168.80.140 sudo docker run -p 8080:8080 -d nani123456789/$JOB_NAME:latest'
                        //   sh "ssh -o StrictHostKeyChecking=no yash@192.168.80.137 ${dockerrm} || true"
                        //   sh "ssh -o StrictHostKeyChecking=no yash@192.168.80.137 ${dockerimg} || true"
                        //   sh "ssh -o StrictHostKeyChecking=no yash@192.168.80.140 sudo docker pull nani123456789/pipeline"
                        //   sh "ssh -o StrictHostKeyChecking=no yash@192.168.80.140 ${dockerrun}"
                       }     
                  }
                }   
            }
             stage('DAST') {
             steps {
                sh 'docker rm dast_baseline || true'
                sh 'docker rm dast_full || true'
                sh 'docker run --name dast_full --network project_project -t owasp/zap2docker-stable zap-full-scan.py -t http://192.168.80.140/LoginWebApp/ || true'
                sh 'docker run --name dast_baseline --network project_project -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.80.140/LoginWebApp/ --autooff || true'
             }
        }
    }
}
