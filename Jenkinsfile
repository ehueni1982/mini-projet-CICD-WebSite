/* import shared libary */
@Library('ehueniapp-slack-share-library')_

pipeline {

    environment {
        IMAGE_NAME = "staticwebsite"
	APP_CONTAINER_PORT = "5000"
	APP_EXPOSED_PORT = "8090"
	IMAGE_TAG = "latest"
	STAGING = "ehueniapp-staging"
	PRODUCTION = "ehueniapp-prod"
	DOCKERHUB_ID = "ehueni1982"
	DOCKERHUB_PASSWORD = credentials('dockerhub_password')
    }
    agent none
    stages {
       stage('Build image') {
           agent any
	   steps {
	      script {
	        sh 'docker build -t ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG .'
	      }
	   }
       }
       stage('Run container based on builded image') {
          agent any
          steps {
             script {
               sh '''
		    echo "Cleaning existing container if exist"
		    docker ps -a | grep -i $IMAGE_NAME && docker rm -f $IMAGE_NAME
		    docker run --name $IMAGE_NAME -d -p $APP_EXPOSED_PORT:$APP_CONTAINER_PORT -e PORT=$APP_CONTAINER_PORT ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
		    docker ps -a | grep -i $IMAGE_NAME
		    sleep 5

               ''' 
	      }

           }
              
        }
        stage('Test image') {
	   agent any
	   steps {
	      script {
	        sh '''
                   #curl localhost | grep -i "Dimension"

	         '''
              }
	      
           }
	        
	}
	stage('Clean container') {
	   agent any
	   steps {
	      script {
	        sh '''
	           docker stop $IMAGE_NAME
	           docker rm $IMAGE_NAME

	         '''
	      }

	   }

	}
        stage ('Login and Push Image on docker hub') {
           agent any
           steps {
	      script {
	        sh '''
	           echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_ID --password-stdin
		   docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG

	        '''
	       }

	    }

         }   
         stage('Push image in staging and deploy it') {
	    when {
		    expression { GIT_BRANCH == 'origin/main' }
	    }
            agent {
		   docker { image 'franela/dind' }
	    }

	    environment {
	         HEROKU_API_KEY = credentials('heroku_api_key')
	    }
	     steps {
	        script {
	          sh '''
	             #apk --no-cache add npm
	             #npm install -g heroku
	             #heroku container:login
		     #heroku create $STAGING || echo "projets already exist" --entrypoint=""
		     #heroku container:push -a $STAGING
		     #heroku container:release -a $STAGING

		  '''
		 }

	      }
	       
	   }
	   stage('Push image in production and deploy it') {
	      when {
	            expression { GIT_BRANCH == 'origin/main' }
	      }

	      agent {
		      docker { image 'franela/dind' }
	      }

              environment {
                  HEROKU_API_KEY = credentials('heroku_api_key')
	      }
	      steps {
	         script {
	           sh '''
		      #apk --no-cache add npm
		      #npm install -g heroku
		      #heroku container:login
		      #heroku create $PRODUCTION || echo "projets already exits" --entrypoint=""
		      #heroku container:push -a $PRODUCTION
		      #"heroku container:release -a $PRODUCTION

		   '''
	          }

	       }   
     		 
           }
       } 
       post {
	     always {

	       script {
		     slackNotifier currentBuild.result
		     
	       }

             }
	     	 
       }
        
  }



