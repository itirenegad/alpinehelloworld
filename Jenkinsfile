pipeline {
  
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "testdvp-2.1"
        CONTAINER_NAME = "alpinehellowolrd"
        USERNAME = "itirenegad"      
        STAGING = "itirenegad-ahw-staging-env"
        PRODUCTION = "itirenegad-ahw-prod-env"
    }
  
    agent none
  
    stages {
     
        stage ('BUILD IMAGE') {
            agent any
            steps {
                script {
                    sh 'docker build -t $USERNAME/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
    
        stage ('RUN TEST CONTAINER') {
            agent any
            steps {
                script {
                    sh  '''
                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true
                        docker run --name $CONTAINER_NAME -d -e PORT=5000 -p 5000:5000 $USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                    '''
                    }
                }
            }
    
        stage ('TEST CONTAINER') {
            agent any
            steps {
                script {
                    sh  '''
                        curl http://localhost:5000 | grep -iq "Hello world!"
                    '''
                }
            }
        }
    
        stage ('CLEAN BUILD ENVIRONMENT AND SAVE ARTEFACT') {
            agent any
            environment {
                PASSWORD = credentials('dockerhub_password')
            }
            steps {
                script {
                    sh  '''
                        docker login -u $USERNAME -p $PASSWORD
                        docker push $USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true
                        docker rmi $USERNAME/$IMAGE_NAME:$IMAGE_TAG
                    '''
                    }
                }
            }
  
        stage ('PUSH IMAGE IN STAGING AND DEPLOY IT') {
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            agent any
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

        stage ('PUSH IMAGE IN PRODUCTION AND DEPLOY IT') {
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            agent any
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
        
      }
}
