pipeline {
    agent any
    stages {
        stage('Lint HTML') {
            steps {
		sudo apt install tidy
		sh 'echo "Lint check..."'
                sh 'tidy -q -e *.html'
                slackSend color: "bad", message: "HTML lint failed!- ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            }
        }
         stage('Lint Dockerfile') {
             steps {
                 script {
                    docker.image('hadolint/hadolint:latest-debian').inside() {
                            sh 'hadolint Dockerfile | tee -a hadolint_lint.txt'
                            sh '''
                                lintErrors=$(stat --printf="%s"  hadolint_lint.txt)
                                if [ "$lintErrors" -gt "0" ]; then
                                    echo "Errors have been found, please see below"
                                    cat hadolint_lint.txt
                                    slackSend color: "bad", message: "Build Failed -${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"

                                    exit 1
                                else
                                    echo "There are no erros found on Dockerfile!!"
                                    slackSend color: "good", message: "Hadolint passed!"
                                fi
                            '''
                    }
                }
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
			--node-ami auto \
			--region us-west-2 \
			--zones us-west-2a \
		        --zones us-west-2b \
			--zones us-west-2c \
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
