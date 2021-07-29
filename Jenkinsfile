pipeline {
    agent any
    stages {
        stage('Lint HTML') {
            steps {
		sh 'echo "Lint check..."'
                sh 'tidy -q -e ./Blue-Container/index.html'
                sh 'tidy -q -e ./Green-Container/index.html'
                echo 'HTML lint successfully completed..'
		sh 'hadolint ./Blue-Container/Dockerfile'
                sh 'hadolint ./Green-Container/Dockerfile'
                echo 'Dockerfile lint successfully completed..'

            }
        }  
	stage('Build and Push Docker Image') {
   	    steps {
                withCredentials([usernamePassword(credentialsId: 'Dockerhub_ID', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
		    sh 'cd ./Blue-Container'
                    sh 'echo "Building Docker Blue-Image..."'
     	    	    sh 'docker build -t tayalsakshi381/capstone-project-blue-image .'
		    sh 'echo "Pushing Docker Blue-Image..."'
     	    	    sh '''
                        docker login -u $USERNAME -p $PASSWORD
			docker push tayalsakshi381/capstone-project-blue-image
                    sh 'cd ..'
	            sh 'echo "Building Docker Green-Image..."'
                    sh 'cd ./Green-Container'
                    sh 'docker build -t tayalsakshi381/capstone-project-green-image .'
		    sh 'echo "Pushing Docker Green-Image..."'
     	    	    sh '''
                        docker login -u $USERNAME -p $PASSWORD
			docker push tayalsakshi381/capstone-project-blue-image
                        docker push tayalsakshi381/capstone-project-green-image
                    '''
		}
            }
        }
	    
	 
	stage('Create k8s cluster') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Create k8s cluster..."'
		    sh '''
			eksctl create cluster \
			--name SakshiKubeCluster \
			--version 1.21 \
			--region us-west-2 \
			--nodegroup-name standard-workers \
			--node-type t2.micro \
			--nodes 2 \
			--nodes-min 1 \
			--nodes-max 3 \
			--managed
		'''
		}
	    }
        }
	    
	stage('Configure kubectl') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Configure kubectl..."'
		    sh 'aws eks --region us-west-2 update-kubeconfig --name SakshiKubeCluster' 
		}
	    }
        }

	stage('Deploy blue container') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Deploy blue container..."'
		    sh 'kubectl apply -f ./Blue-Container/blue-controller.json'
		}
	    }
	}
	    
	stage('Deploy green container') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Deploy green container..."'
		    sh 'kubectl apply -f ./Green-Container/green-controller.json'
		}
	    }
	}
	    
	stage('Create blue service') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Create blue service..."'
		    sh 'kubectl apply -f ./Blue-Container/blue-service.json'
		}
	    }
	}
       stage('Wait for user to approve') {
            steps {
                input "Redirect traffic to green?"
            }
        }
	    
	stage('Update service to green') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Update service to green..."'
		    sh 'kubectl apply -f ./Green-Container/green-service.json'
		}
	    }
         }
	
        stage('Send Slack Notification'){ 
            steps{
                slackSend color: "good", message: " ${currentBuild.fullDisplayName} has completed successfully!"
            }
        }

     }   
 }

