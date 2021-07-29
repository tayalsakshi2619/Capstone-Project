pipeline {
    agent any
    stages {
        stage('Lint HTML') {
            steps {
		sh 'echo "Lint check..."'
                sh 'tidy -q -e *.html'
		sh 'hadolint Dockerfile'
            }
        }  
	stage('Build Docker Image') {
   	    steps {
                withCredentials([usernamePassword(credentialsId: 'Dockerhub_ID', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
		    sh 'echo "Building Docker Image..."'
     	    	    sh 'docker build -t tayalsakshi381/capstone-project .'
		}
            }
        }
	    
	stage('Push Image To Dockerhub') {
   	    steps {
                withCredentials([usernamePassword(credentialsId: 'Dockerhub_ID', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
		    sh 'echo "Pushing Docker Image..."'
     	    	    sh '''
                        docker login -u $USERNAME -p $PASSWORD
			docker push tayalsakshi381/capstone-project
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
			--version 1.14 \
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
		    sh 'kubectl apply -f blue.yaml'
		}
	    }
	}
	    
	stage('Deploy green container') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Deploy green container..."'
		    sh 'kubectl apply -f green.yaml'
		}
	    }
	}
	    
	stage('Create blue service') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Create blue service..."'
		    sh 'kubectl apply -f blue_service.yaml'
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
		    sh 'kubectl apply -f green_service.yaml'
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
