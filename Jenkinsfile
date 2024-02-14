pipeline {
    agent { label 'jenkins-agent' }
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
	APP_NAME = "register-app-pipeline"
 	RELEASE = "1.0.0"
    	DOCKER_USER = "arjillibalaji"
    	DOCKER_PASS = 'dockerhub-cred'
    	IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
    	IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/balajiarjilli/register-app'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }

        stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-cred') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }

	stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-cred'
                }	
            }
        }

	stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
       }

	    
         
   }
}
