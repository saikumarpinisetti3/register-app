pipeline {
    agent any

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "saikumarpinisetti"
        DOCKER_PASS = 'Supershot#143'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("Devops")
    }

    stages {
        stage('Checkout from SCM') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/saikumarpinisetti3/register-app.git'
                }
            }
        }

        stage('Build Application') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('Test Application') {
            steps {
                sh "mvn test"
            }
        }

        stage('Functional Testing') {
            steps {
                script {
                    sh "mvn verify"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'Devops') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker image build -t ${APP_NAME}:${IMAGE_TAG} ."
                    sh "docker image tag ${APP_NAME}:${IMAGE_TAG} ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"
                    sh "docker image tag ${APP_NAME}:${IMAGE_TAG} ${DOCKER_USER}/${APP_NAME}:latest"
                }
            }
        }

        stage('Docker image push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "docker login -u $DOCKER_USER -p $DOCKER_PASS"
                        sh "docker image push $DOCKER_USER/$APP_NAME:$IMAGE_TAG"
                        sh "docker image push $DOCKER_USER/$APP_NAME:latest"
                    }
                }
            }
        }

        stage('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                   cat deployment.yaml
                   sed -i 's|${APP_NAME}.*|${APP_NAME}:${IMAGE_TAG}|g' deployment.yaml
                   cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                   git config --global user.name "saikumarpinisetti3"
                   git config --global user.email "saikumarpinisetti3@gmail.com"
                   git add deployment.yaml
                   git commit -m "Updated Deployment Manifest"
                """
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'pass', usernameVariable: 'user')]){
                    sh "git push https://github.com/saikumarpinisetti3/register-app main"
                }
            }
        }
    }
}
