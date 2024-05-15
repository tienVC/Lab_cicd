pipeline {
    agent any
    tools {
        jdk "jdk17"
        nodejs "node16"
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "Lab_cicd"
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
                withSonarQubeEnv('Sonar-Qube Sever') {
                    sh '''/opt/sonarscanner/bin/sonar-scanner -Dsonar.projectName=Lab_cicd \
                    -Dsonar.projectKey=Lab_cicd '''
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
	 stage ('Cleanup Artifacts') {
             steps {
                 script {
                      sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                      sh "docker rmi ${IMAGE_NAME}:latest"
                 }
             }
         }
	 stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://3.132.221.232:8080//job/Lab_cicd/buildWithParameters?token=gitops-token'"
                }
            }
         }
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
