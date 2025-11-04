pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "19862020" // replace this with your docker-id
MOVIE_DOCKER_IMAGE = "movie-image"
CAST_DOCKER_IMAGE = "cast-image"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
// App environment variables
DATABASE_URI = "postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev"
CAST_SERVICE_HOST_URL = "http://cast_service:8000/api/v1/casts/"
POSTGRES_USER= "movie_db_username"
POSTGRES_PASSWORD= "movie_db_password"
POSTGRES_DB= "movie_db_dev"
        
}
agent any // Jenkins will be able to select all available agents
stages {
        stage('Docker Build'){ // docker build image stage 
            steps {
                script {
                sh '''
                 docker rm -f movie-container
                 echo "Building Movie image"
                 docker build -t $DOCKER_ID/$MOVIE_DOCKER_IMAGE:$DOCKER_TAG ./movie-service
                 echo "Building Cast image"
                 docker build -t $DOCKER_ID/$CAST_DOCKER_IMAGE:$DOCKER_TAG ./cast-service
                 

                sleep 6
                '''
                }
            }
        }
    
        stage('Start Movie DB') {
    steps {
        script {
            sh '''
            docker rm -f movie_db 

            docker run -d \
                --name movie_db \
                -e POSTGRES_USER=POSTGRES_USER \
                -e POSTGRES_PASSWORD=POSTGRES_PASSWORD \
                -e POSTGRES_DB=POSTGRES_DB \
                -v postgres_data_movie:/var/lib/postgresql/data/ \
                postgres:12.1-alpine
            sleep 10
            '''
        }
    }
}
        stage('Docker run'){ // run container from our built image
                steps {
                    script {
                    sh '''

                    docker run -d \
                        -e DATABASE_URI=$DATABASE_URI \
                        -e CAST_SERVICE_HOST_URL=$CAST_SERVICE_HOST_URL \
                        -v ./movie-service/:/app/ \
                        -p 8001:8000 \
                        --name movie-container \
                        $DOCKER_ID/$MOVIE_DOCKER_IMAGE:$DOCKER_TAG \
                        "uvicorn" "app.main:app" "--host" "0.0.0.0" "--port" "8000" "--loop" "asyncio" "--workers" "1"
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
