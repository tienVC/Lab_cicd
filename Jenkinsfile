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
        // stage("Quality Gate") {
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'sqp_c9d5aa06e36b6816005c9dc5368c9b5c8de25f29'
        //         }
        //     }
        // }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
	        stage('Build Images') {
	            steps {
	                sh 'docker-compose build'
	            }
	        }
	        stage('Push Images') {
	            steps {
	                sh 'docker-compose push'
	            }
	        }
	 stage ('Cleanup Artifacts') {
             steps {
                 script {
                      // sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                      // sh "docker rmi ${IMAGE_NAME}:latest"
			  sh 'rm -rf target/*'
                 }
             }
         }
	 // stage("Trigger CD Pipeline") {
  //           steps {
  //               script {
  //                   sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://3.132.221.232:8080//job/Lab_cicd/buildWithParameters?token=gitops-token'"
  //               }
  //           }
  //        }
    }
     // post {
     //    always {
     //       emailext attachLog: true,
     //           subject: "'${currentBuild.result}'",
     //           body: "Project: ${env.JOB_NAME}<br/>" +
     //               "Build Number: ${env.BUILD_NUMBER}<br/>" +
     //               "URL: ${env.BUILD_URL}<br/>",
     //           to: 'vucongtien0311@gmail.com',                              
     //    }
     // }
}  
