pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "19862020" // replace this with your docker-id
MOVIE_DOCKER_IMAGE = "movie-image"
CAST_DOCKER_IMAGE = "cast-image"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage('Docker Build Movie Service'){ // docker build image stage 
            steps {
                script {
                sh '''
                 docker rm -f movie-container
                 docker build -t $DOCKER_ID/$MOVIE_DOCKER_IMAGE:$DOCKER_TAG ./movie-service

                sleep 6
                '''
                }
            }
        }
    
        stage('Docker run Movie Service'){ // run container from our built image
                steps {
                    script {
                    sh '''
                    docker run -d -p 8001:8000 --name movie-container $DOCKER_ID/$MOVIE_DOCKER_IMAGE:$DOCKER_TAG 

                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost:8001
                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
                }
            }

        }



}

}
