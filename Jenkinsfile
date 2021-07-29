pipeline {
    agent any
    stages {
        stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-west-2', credentials:'AWSCredentials') {
					sh '''
						kubectl config use-context arn:aws:eks:us-west-2:982828900997:cluster/SakshiKubeCluster
					'''
				}
			}
		}


	stage('Deploy blue container') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Deploy blue container..."'
		    sh 'kubectl apply -f ./blue.yaml'
		}
	    }
	}
	    
	stage('Deploy green container') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Deploy green container..."'
		    sh 'kubectl apply -f ./green.yaml'
		}
	    }
	}
	    
	stage('Create blue service') {
	    steps {
		withAWS(credentials: 'AWSCredentials', region: 'us-west-2') {
		    sh 'echo "Create blue service..."'
		    sh 'kubectl apply -f ./blue_service.yaml'
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
		    sh 'kubectl apply -f ./green_service.yaml'
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
