pipeline {
    agent any
    tools {
        jdk "jdk17"
        nodejs "node16"
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "lab_cicd"
        RELEASE = "1.0.0"
        DOCKER_USER = "tientrang0311"
        DOCKER_PASS = "dockerhub"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
	DOCKERHUB_API_TOKEN = credentials("docker_access_token")
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/tienVC/Lab_cicd.git'
            }
        }
         stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('SonaQube-Sever') {
                     sh '/opt/sonar-scanner/bin/sonar-scanner -Dsonar.projectKey=lab_cicd   -Dsonar.sources=.   -Dsonar.host.url=http://54.215.81.76:9000   -Dsonar.login=sqp_c9d5aa06e36b6816005c9dc5368c9b5c8de25f29 '
                }
            }

        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
	    stage("Build & Push Docker Image") {
             steps {
                 script {
                     docker.withRegistry('',DOCKER_PASS) {
                         sh 'docker-compose build && docker-compose push'
                     }
                 }
             }
         }
	        // stage('Build Images') {
	        //     steps {
	        //         sh 'docker-compose build'
	        //     }
	        // }
	        // stage('Push Images') {
	        //     steps {
	        //         sh 'docker-compose push'
	        //     }
	        // }
	 // stage ('Cleanup Artifacts') {
  //            steps {
  //                script {
  //                     // sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
  //                     // sh "docker rmi ${IMAGE_NAME}:latest"
		// 	  sh 'rm -rf target/*'
  //                }
  //            }
  //        }
	 // stage("Trigger CD Pipeline") {
  //           steps {
  //               script {
  //                   sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://3.132.221.232:8080//job/Lab_cicd/buildWithParameters?token=gitops-token'"
  //               }
  //           }
  //        }
    }
     telegramNotify(
	  token: credentials('TELEGRAM_BOT_TOKEN'), // Reference credentials containing Bot Token
	  chatId: '93372553', // Replace with your actual Chat ID
	  message: 'Your build '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) is ${currentBuild.result}'
	)
}  
