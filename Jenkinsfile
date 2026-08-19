pipeline {
    agent any

    environment {
        DOCKER_USERNAME     = credentials('DOCKER_USERNAME')
        DOCKER_ACCESS_TOKEN = credentials('DOCKER_ACCESS_TOKEN')
        GIT_REPO            = credentials('GIT_REPO')
        DOCKER_IMAGE        = 'jenkins-demo-image'
        IMAGE_TAG           = "${env.BUILD_NUMBER}"
        CONTAINER_NAME      = 'jenkins-demo'
    }

    stages {
        stage('SCM') {
            steps {
                git url: "${env.GIT_REPO}", branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Using single quotes '$VAR' resolves the Jenkins secret-interpolation warning
                sh 'docker image build -t $DOCKER_USERNAME/$DOCKER_IMAGE:$IMAGE_TAG -t $DOCKER_USERNAME/$DOCKER_IMAGE:latest .'
            }
        }
        stage('Push Docker Image to DockerHub'){
            steps {
                // Login and Push image to Dockerhub
                   sh '''
                        echo "$DOCKER_ACCESS_TOKEN" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push $DOCKER_USERNAME/$DOCKER_IMAGE:$IMAGE_TAG
                        docker push $DOCKER_USERNAME/$DOCKER_IMAGE:latest
                        docker logout

                    '''
                                
    
            }
        }

        stage('Run Docker Container'){
            steps{
                sh '''
                docker rm -f $CONTAINER_NAME || true 
                docker container run -d --name $CONTAINER_NAME -p 9090:80 $DOCKER_USERNAME/$DOCKER_IMAGE:latest
                sleep 5
                docker ps --filter name='$CONTAINER_NAME'
                curl -f http://localhost:9090
                '''
            }
        }
    }


// Post Actions

    post {
        always {
            sh 'docker rmi -f $DOCKER_USERNAME/$DOCKER_IMAGE:$IMAGE_TAG $DOCKER_USERNAME/$DOCKER_IMAGE:latest || true'
        }


//Post Action End
    }


//Pipeline End    
}
