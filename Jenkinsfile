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
 stage('Build & Push Docker Image') {
    steps {
        script {
            docker.withRegistry('https://registry.example.com', 'Supershot#143') {
                def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                dockerImage.push("${IMAGE_TAG}")
                dockerImage.push('latest')
            }
        }
    }
}

       stage('image scanning'){
            steps{
                script{

                   sh 'trivyimage saikumarpinisetti/register-app-pipeline:latest > ss.txt'
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
