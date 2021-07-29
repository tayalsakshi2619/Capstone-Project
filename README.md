Udacity Cloud DevOps Engineer Capstone Project
In this project I have applied the skills and knowledge which were developed throughout the Cloud DevOps Nanodegree program. These include:

 - Working in AWS
 - Using Jenkins or Circle CI to implement Continuous Integration and Continuous Deployment
 - Building pipelines
 - Working with Ansible and CloudFormation to deploy clusters
 - Building Kubernetes clusters
 - Building Docker containers in pipelines
 
 Following are the guidelines to deploy to Kubernetes Cluster:
 
 1. Setup an EC2 instance with following requirements:
    - Jenkins
    - AWS-CLI
    - Kubectl
    - EKCSTL
    - Docker
    - Tidy and Hadolint

2. After, downloading above requirements, we can configure following plugins on Jenkins.
    - BlueOcean - For Better GUI
    - CloudBees - For managing AWS credentials
    - AWS-Pipeline
    - Docker
    - Slack Notification
   
3. Store your AWS and Docker credentials in Jenkins.
4. Set up a webhook url to connect your Git repo with Jenkins
5. Configure slack so that you can receive notifications about build.
6. After the configuration is complete, we can go ahead to build and deploy our dockerized app to AWS Kubernetes Cluster.
          The pipeline looks like this:
          
          
  
