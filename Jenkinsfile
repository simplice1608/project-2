def getDockerTag(){
        def tag = sh script: 'git rev-parse HEAD', returnStdout: true
        return tag
        }

pipeline {
    agent {label 'node1'}
    environment{
	    Docker_tag = getDockerTag()
        AWS_DEFAULT_REGION="us-east-1"
        }
    tools { 
        maven '3.9.0' 
    }
    stages {
        stage('Checkout git') {
            steps {
               git branch: 'main', url: 'https://github.com/simplice1608/project-2.git'
            }
        }
        
        stage ('Build & JUnit Test') {
            steps {
                sh 'mvn package' 
            }
            post {
               success {
                    junit 'target/surefire-reports/**/*.xml'
                }   
            }
        }

        stage("ArchiveArtifacts"){

            steps{
               archiveArtifacts artifacts: 'target/myapp-1.0.jar', followSymlinks: false
            }
	}
        stage('SonarQube Analysis'){
            steps{
                withSonarQubeEnv('sonar') {
                        sh "mvn sonar:sonar"
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        
        stage('Docker Build & Push') {
            steps {
      	        sh 'docker build -t simplice1608/sprint-boot-app:$Docker_tag .'
                withCredentials([string(credentialsId: 'docker', variable: 'docker_password')]) {		    
				  sh 'docker login -u simplice1608 -p $docker_password'
				  sh 'docker push simplice1608/sprint-boot-app:$Docker_tag'
			}
            }
        }
        stage('Image Scan') {
            steps {
      	        sh ' trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o report.html simplice1608/sprint-boot-app:$Docker_tag '
            }
        }
        stage('Upload Scan report to AWS S3') {
            //   steps {
            //       sh 'aws s3 cp report.html s3://project-2/'
            //   }
            steps {
              withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                sh '''
                  aws s3 cp report.html s3://pipeline-project-2/
                   '''
        }
         }
        }
        
        stage("Approval"){

        steps{

            timeout(time: 15, unit: 'MINUTES'){ 
	               input message: 'Do you approve deployment for production?' , ok: 'Yes'}

        }
       }

        stage ('Prod pre-request') {
            agent { label 'node1' }
			steps{
			 	script{
				    sh '''final_tag=$(echo $Docker_tag | tr -d ' ')
				     echo ${final_tag}test
				     sed -i "s/docker_tag/$final_tag/g"  spring-boot-deployment.yaml
				     '''
				}
			}
        }
        stage('Deploy to k8s') {
            agent { label 'node1' }
              steps {
                script{
                    kubernetesDeploy configs: 'spring-boot-deployment.yaml', kubeconfigId: 'kubernetes'
                }
            }
        }
        
 
    }
    post{
        always{
            sendSlackNotifcation()
            }
        }
}
def sendSlackNotifcation()
{
    if ( currentBuild.currentResult == "SUCCESS" ) {
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *SUCCESS*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL} \n"
        slackSend( channel: "#general", token: 'slack', color: 'good', message: "${buildSummary}")
    }
    else {
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *FAILURE*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL}\n  \n "
        slackSend( channel: "#general", token: 'slack', color : "danger", message: "${buildSummary}")
    }
}

    
