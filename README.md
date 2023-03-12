# DevSecOps-Project

In this project, end-to-end CI/CD pipeline while keeping in mind Securities Best Practices, DevSecOps principles and used all these tools *Git, GitHub , Jenkins,Maven, Junit, SonarQube, Docker, Trivy, AWS S3, Docker Hub, Kubernetes , Slack*  to achive the goal.


## Project Architecture
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/architecture.png)

## Pipeline flow:
1. Jenkins will fetch the code from the remote repo or you can setup webhook with github project so that Jenkins job trigger after every commit in main branch.
2. Maven will build the code, if the build fails, the whole pipeline will become a failure and Jenkins will notify the user, If build success then 
3. Junit will do unit testing, if the application passes test cases then will go to the next step otherwise the whole pipeline will become a failureJenkins will notify the user that your build fails.
4.SonarQube scanner will scan the code and will send the report to the SonarQube server, where the report will go through the quality gate and gives the output to the web Dashboard. 
                        In the quality gate, we define conditions or rules like how many bugs or vulnerabilities, or code smells should be present in the code. 
 Also, we have to create a webhook to send the status of quality gate status to Jenkins.
 If the quality gate status becomes a failure, the whole pipeline will become a failure then Jenkins will notify the user that your build fails.
5. After the quality gate passes, Docker will build the docker image.
if the docker build fails when the whole pipeline will become a failure and Jenkins will notify the user that your build fails.
6. Trivy will scan the docker image, if it finds any Vulnerability then the whole pipeline will become a failure, and the generated report will be sent to s3 for future review and Jenkins will notify the user that your build fails.
7. After trivy scan docker images will be pushed to the docker hub, if the docker fails to push docker images to the docker hub then the pipeline will become a failure and Jenkins will notify the user that your build fails.
8. After the docker push, Jenkins will create deployment and service in minikube and our application will be deployed into Kubernetes.
if Jenkins fails to create deployment and service in Kubernetes, the whole pipeline will become a failure and Jenkins will notify the user that your build fails.


### PreRequisites
1. JDK 
1. Git 
1. Github
1. Jenkins
1. Sonarqube
1. Docker
1. Trivy
1. AWS account
1. Docker Hub account
1. kubernetes cluster & Kubectl
1. Slack