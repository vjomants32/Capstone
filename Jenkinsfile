  
pipeline {
	agent any
	stages {
		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e index.html'
			}
		}
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerHub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						sudo docker build -t vjomants/valdiscapstone .
					'''
				}
			}
		}
		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerHub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						sudo docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						sudo docker push vjomants/valdiscapstone
					'''
				}
			}
		}
		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-east-2', credentials:'MyCredentials') {
					sh '''
						sudo kubectl config get-contexts
						sudo kubectl config use-context arn:aws:eks:us-east-2:839035252841:cluster/valdiscapstone
					'''
				}
			}
		}
		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-east-2', credentials:'MyCredentials') {
					sh '''
						sudo kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}
		stage('Deploy green container') {
			steps {
				withAWS(region:'us-east-2', credentials:'MyCredentials') {
					sh '''
						sudo kubectl apply -f ./green-controller.json
					'''
				}
			}
		}
		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-east-2', credentials:'MyCredentials') {
					sh '''
						sudo kubectl apply -f ./blue-service.json
					'''
				}
			}
		}
		stage('Wait user approve') {
		    steps {
			input "Ready to redirect traffic to green?"
            }
        }
		stage('Create the service in the cluster, redirect to green') {
			steps {
				withAWS(region:'us-east-2', credentials:'MyCredentials') {
					sh '''
						sudo kubectl apply -f ./green-service.json
					'''
				}
			}
		}

	}
}
