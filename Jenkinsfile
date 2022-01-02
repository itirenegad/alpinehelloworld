pipeline {
  
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "testdvp-4.0"
        CONTAINER_NAME = "alpinehellowolrd"
        USERNAME = "itirenegad"      
        STAGING = "itirenegad-ahw-staging-env"
        PRODUCTION = "itirenegad-ahw-prod-env"
        EC2_PRODUCTION_HOST = "34.243.75.9"
    }
  
    agent none
  
    stages {
     
        stage ('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t $USERNAME/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
    
        stage ('Run test container') {
            agent any
            steps {
                script {
                    sh  '''
                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true
                        docker run -d --name $CONTAINER_NAME -e PORT=5000 -p 5000:5000 $USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                    '''
                }
            }
        }
    
        stage ('Test application') {
            agent any
            steps {
                script {
                    sh  '''
                        curl http://localhost:5000 | grep -iq "Hello world!"
                    '''
                }
            }
        }
    
        stage ('Clean env and save artifact') {
            agent any
            environment {
                PASSWORD = credentials('dockerhub_password')
            }
            steps {
                script {
                    sh  '''
                        docker login -u $USERNAME --password $PASSWORD
                        docker push $USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true
                        docker rmi $USERNAME/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
  
        stage ('Push image in staging and deploy it') {
            agent any
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                        heroku container:login
                        heroku create $STAGING || echo "PROJECT ALREADY EXISTS" 
                        heroku container:push -a $STAGING web
                        heroku container:release -a $STAGING web
                    '''
                }
            }
        } 

        stage ('Push image in Prod and deploy it') {
            agent any
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                        heroku container:login
                        heroku create $PRODUCTION || echo "PROJECT ALREADY EXISTS" 
                        heroku container:push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web
                    '''    
                }   
            }
        }

        stage ('Deploy app on EC2-cloud Production') {
            agent any
            when{
               expression { GIT_BRANCH == 'origin/master'}
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "ec2_prod_private_key", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script{ 

                            timeout(time: 15, unit: "MINUTES") {
                                input message: 'Do you want to approve the deploy in production?', ok: 'Yes'
                            }

                            sh '''
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker run -d --name alpinehelloword -e PORT=5000 -p 5000:5000 $USERNAME/$IMAGE_NAME:$IMAGE_TAG
                               '''
                        }
                    }
                }
            }
        }
    }   

    post {
        success {
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
}
