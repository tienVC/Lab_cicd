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
                    sh '''/opt/sonar-scanner-4.8.0.2856-linux/bin/sonar-scanner -Dsonar.projectName=lab_cicd \
                    -Dsonar.projectKey=lab_cicd '''
                }
            }

        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
	// stage("Build & Push Docker Image") {
	//              steps {
	//                  script {
	//                      docker.withRegistry('',DOCKER_PASS) {
	//                          docker_image = docker.build "${IMAGE_NAME}"
	//                      }
	//                      docker.withRegistry('',DOCKER_PASS) {
	//                          docker_image.push("${IMAGE_TAG}")
	//                          docker_image.push('latest')
	//                      }
	//                  }
	//              }
	//          }

        stage("Deploy with Docker Compose") {
            steps {
                script {
                    sh "docker-compose -f docker-compose.yml up -d"
                }
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
