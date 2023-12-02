pipeline {
    agent any
    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "saikumarpinisetti"
            DOCKER_PASS = 'Supershot#143'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    JENKINS_API_TOKEN = credentials("Devops")
    }
	stages{
        stage('Checkout from SCM'){
                steps {
                    git branch: 'main', url: 'https://github.com/saikumarpinisetti3/register-app.git'
                }
        }

        stage('Build Application'){
            steps {
                sh "mvn clean package"
            }

       }

       stage('Test Application'){
           steps {
                 sh "mvn test"
           }
       }
       stage('functional testing'){
            steps{
                script{
                    sh "mvn verify"
                }
            }
        }

       stage('SonarQube Analysis'){
           steps {
	           script {
		      withSonarQubeEnv(credentialsId: 'Devops') {
    // some block
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }
 stage('Build Docker Image') {
    steps {
        script {
	sh 'docker image build -t $APP_NAME:v1.$BUILD_ID .' 
	 sh 'docker image tag $APP_NAME:v1.$BUILD_ID $DOCKER_USER/$APP_NAME:v1.$BUILD_ID'
	sh 'docker image tag $APP_NAME:v1.$BUILD_ID $DOCKER_USER/$APP_NAME:latest'
        }
    }
}
	stage('Docker image push'){

             steps{

              script{
                   withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                     
                     sh 'docker login -u $DOCKER_USER -p ${DOCKER_PASS}'
                     sh 'docker image push $DOCKER_USER/$APP_NAME:v1.$BUILD_ID'
                     sh 'docker image push $DOCKER_USER/$APP_NAME:latest'
                  }
                }
             }
        }      

       stage('image scanning'){
            steps{
                script{

                   sh 'docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ashfaque9x/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table > ss.txt'
                   sh 'cat ss.txt'
                }
            }
        }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
            }
       }
}
