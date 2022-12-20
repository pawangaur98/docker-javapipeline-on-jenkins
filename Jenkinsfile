pipeline {
	agent {	
		label 'slave1'
		}
	stages {
		stage("SCM") {
			steps {
				git branch: 'main', url: 'https://github.com/pawangaur98/docker-javapipeline-on-jenkins.git'
				}
			}

		stage("build") {
			steps {
				sh 'sudo mvn dependency:purge-local-repository'
				sh 'sudo mvn clean package'
				}
			}
		stage("Image") {
			steps {
				sh 'sudo docker build -t java-repo:$BUILD_TAG .'
				sh 'sudo docker tag java-repo:$BUILD_TAG pawangaur/pipeline-java:$BUILD_TAG'
				}
			}
				
	
		stage("Docker Hub") {
			steps {
			withCredentials([string(credentialsId: 'docker_hub', variable: 'docker_hub_password_var')])    {
				sh 'sudo docker login -u pawangaur -p ${docker_hub_password_var}'
				sh 'sudo docker push pawangaur/pipeline-java:$BUILD_TAG'
				}
			}
		}

		stage("QAT Testing") {
			steps {
//				sh ' docker rm -f $(docker ps -a -q)'
				sh 'sudo docker run -dit -p 8082:8080 pawangaur/pipeline-java:$BUILD_TAG'
				}
			}
	 	stage("testing website") {
			steps {
				retry(8) {
				sh "curl --silent http://3.109.158.89:8082/java-web-app/ | grep 'India'"
				}
	   		}
		}

		stage("Approval status") {
			steps {
				script {  
                		Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                		echo 'userInput: ' + userInput
				}
	 		}
		}
		
		stage("Prod Env") {
			steps {
			 sshagent(['ssh-credentials']) {
			    sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.109.158.89 sudo docker rm -f $(sudo docker ps -a -q)' 
	                    sh "ssh -o StrictHostKeyChecking=no ubuntu@3.109.158.89 sudo docker run  -d  -p  49153:8080  pawangaur/pipeline-java:$BUILD_TAG"
				}
			}
		}
    	}
}
