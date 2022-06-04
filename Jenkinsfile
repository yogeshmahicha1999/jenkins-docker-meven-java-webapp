pipeline {
    
    agent { 
        label "linuxbuildnode"
    }
    
    stages {
        stage('SCM') {
            steps {
                git 'https://github.com/yogeshmahicha1999/jenkins-docker-meven-java-webapp.git'
            }
        }
        
        stage('Build by Maven Package') {
            steps {
                sh 'mvn clean package' 
            }
        }
        
        stage('Build Docker OWN image') {
            steps {
                sh "sudo docker build -t yogeshkumarmahicha/javaweb:${BUILD_TAG}  ." 
            }
        }
        
        stage('Push Image to Docker HUB') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_PWD', variable: 'DOCKER_HUB_PASS_CODE')]) {
    // some block
                sh "sudo docker login -u yogeshkumarmahicha -p $DOCKER_HUB_PASS_CODE"
                }
                
                sh "sudo docker push yogeshkumarmahicha/javaweb:${BUILD_TAG}"
            }
        }
    
        stage('Deploy webApp in DEV Env') {
            steps {
                sh 'sudo docker rm -f myjavaapp1'
                sh "sudo docker run -d -p 8080:8080  --name myjavaapp1   yogeshkumarmahicha/javaweb:${BUILD_TAG}" 
            }
        }
        
        stage('Deploy webApp in QA/Test Env') {
            steps {
                sshagent(['QA_ENV-SSH_CRED']) {
                
                sh "ssh -o StrictHostKeyChecking=no ec2-user@13.235.104.8 sudo docker rm -f myjavaapp1"
                sh "ssh ec2-user@13.235.104.8 sudo docker run -d -p 8080:8080  --name myjavaapp1   yogeshkumarmahicha/javaweb:${BUILD_TAG}" 
                }
            }
        }
    
        stage('QAT Test') {
            steps {
                
               
                
                retry(10) {
                    sh 'curl --silent http://13.235.104.8:8080/java-web-app/ |  grep India'
                }
            
               
            }
        }

        stage('approved'){
            steps{
                script {
                    Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                    echo 'userInput: ' + userInput

                    if(userInput == true) {
                    // do action
                    } else {
                        // not do action
                        echo "Action was aborted."
                    }
            
                
            }
            }
        }
        
        
        stage('Deploy webApp in Prod Env') {
            steps {
                sshagent(['QA_ENV-SSH_CRED']) {
                
                sh "ssh -o StrictHostKeyChecking=no ec2-user@3.109.184.35 sudo docker rm -f myjavaapp1"
                sh "ssh ec2-user@3.109.184.35 sudo docker run -d -p 8080:8080  --name myjavaapp1   yogeshkumarmahicha/javaweb:${BUILD_TAG}" 
                }
            }
        }
        
    }
}
