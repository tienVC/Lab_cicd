pipeline {
    agent any
    tools {
        jdk "jdk17"
        nodejs "node16"
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "multi-client"
        RELEASE = "1.0.0"
        DOCKER_USER = "tientrang0311"
        DOCKER_PASS = "dockerhub"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
	DOCKERHUB_API_TOKEN = credentials("docker_access_token")
	telegram_bot=credentials('TELEGRAM_BOT_TOKEN')
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
                         sh 'docker-compose build --build-arg IMAGE_TAG=${IMAGE_TAG} && docker-compose push'
                     }
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
	    stage("Cleanup Workspace") {
             steps {
                cleanWs()
             }
         }
         stage("Checkout from SCM") {
             steps {
                     git branch: 'main', credentialsId: 'github', url: 'https://github.com/tienVC/gitops'
             }
         }
         stage("Update the Deployment Tags") {
            steps {
                sh """
                    cat client-deployment.yml
                    sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' client-deployment.yml
                    cat client-deployment.yml
                """
            }
         }
         stage("Push the changed deployment file to GitHub") {
            steps {
                sh """
                    git config --global user.name "tienvc"
                    git config --global user.email "vucongtien0311@gmail.com"
                    git add client-deployment.yml
                    git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh "git push https://github.com/tienVC/gitops main"
                }
            }
         }

    }
	post {
	        success {
	            script {
	                // bat "curl -X POST -H \"Content-Type: application/json\" -d \"{\\\"chat_id\\\":${CHAT_ID}, \\\"text\\\": \\\"Pipeline succeeded!\\\", \\\"disable_notification\\\": false}\" https://api.telegram.org/bot${TOKEN}/sendMessage"
			  // sh "curl -s -X POST https://api.telegram.org/bot${telegram_bot}/sendMessage -d chat_id=1936482774 -d text="sucess""
			     sh """
			        curl -X POST \
			                -d chat_id=1936482774 \
			                -d parse_mode=HTML \
			                -d text='<b>success</b>' \
			                https://api.telegram.org/bot${telegram_bot}/sendMessage
			        """
	            }
	        }
	        failure {
	            script {
	              	// sh """curl -s -X POST https://api.telegram.org/bot${telegram_bot}/sendMessage -d chat_id=1936482774 -d text="fail""""
			    // bat "curl -X POST -H \"Content-Type: application/json\" -d \"{\\\"chat_id\\\":${CHAT_ID}, \\\"text\\\": \\\"Pipeline failed!\\\", \\\"disable_notification\\\": false}\" https://api.telegram.org/bot${TOKEN}/sendMessage"
		             sh """
				        curl -X POST \
				                -d chat_id=${chatid} \
				                -d parse_mode=HTML \
				                -d text='<b>false</b>' \
				                https://api.telegram.org/bot${telegram_bot}/sendMessage
				        """
		    }
	        }
	    }

}  
