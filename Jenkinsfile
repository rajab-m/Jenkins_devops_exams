pipeline { 
environment { // Declaration of environment variables gg
DOCKER_ID="rajabm1311" // replace this with your docker-id
MOVIE_DOCKER_IMAGE = "movie-image"
CAST_DOCKER_IMAGE = "cast-image"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
// App environment variables
DATABASE_URI="postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev"
DATABASE_CAST_URI="postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev"
CAST_SERVICE_HOST_URL="http://cast_service:8000/api/v1/casts/"
POSTGRES_USER="movie_db_username"
POSTGRES_PASSWORD="movie_db_password"
POSTGRES_DB="movie_db_dev"
CAST_POSTGRES_USER="cast_db_username"
CAST_POSTGRES_PASSWORD="cast_db_password"
CAST_POSTGRES_DB="cast_db_dev"
DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve docker password from secret text called docker_hub_pass saved on jenkins     
KUBECONFIG = credentials("config")
}
agent any // Jenkins will be able to select all available agents
stages {
        stage('Docker Build: Movie image and Casts image'){ // docker build image stage 
            steps {
                script {
                sh '''
                 docker login -u $DOCKER_ID -p $DOCKER_PASS
                 #docker rm -f movie-service
                 echo "Building Movie-service image"
                 docker build -t $DOCKER_ID/$MOVIE_DOCKER_IMAGE:$DOCKER_TAG ./movie-service
                 echo "Building Cast image"
                 docker build -t $DOCKER_ID/$CAST_DOCKER_IMAGE:$DOCKER_TAG ./cast-service
                 
                 

                sleep 6
                '''
                }
            }
        }
    
        stage('Docker run Movie DB container') {
            steps {
                script {
                    sh '''
                    docker network create movie_net || true
                    #docker rm -f movie_db 
                    #docker volume rm postgres_data_movie   # important , if we don't delete it the old credentials and env variables will be used
                    docker run -d \
                        --name movie_db \
                        --network movie_net \
                        -e POSTGRES_USER=$POSTGRES_USER \
                        -e POSTGRES_PASSWORD=$POSTGRES_PASSWORD \
                        -e POSTGRES_DB=$POSTGRES_DB \
                        -v postgres_data_movie:/var/lib/postgresql/data/ \
                        postgres:12.1-alpine
                    sleep 10
                    '''
                }
            }
        }
        stage('Docker run Cast DB container') {
            steps {
                script {
                    sh '''
                    #docker rm -f cast_db 
                    #docker volume rm postgres_data_cast   # important , if we don't delete it the old credentials and env variables will be used
                    docker run -d \
                        --name cast_db \
                        --network movie_net \
                        -e POSTGRES_USER=$CAST_POSTGRES_USER \
                        -e POSTGRES_PASSWORD=$CAST_POSTGRES_PASSWORD \
                        -e POSTGRES_DB=$CAST_POSTGRES_DB \
                        -v postgres_data_cast:/var/lib/postgresql/data/ \
                        postgres:12.1-alpine
                    sleep 10
                    '''
                }
            }
        }
        stage('Docker run Movie-service container'){ // run container from our built image
                steps {
                    script {
                    sh """
                        docker run -d \
                          -e DATABASE_URI=$DATABASE_URI \
                          -e CAST_SERVICE_HOST_URL=$CAST_SERVICE_HOST_URL \
                          -v ./movie-service/:/app/ \
                          -p 8001:8000 \
                          --name movie-service \
                          --network movie_net \
                          $DOCKER_ID/$MOVIE_DOCKER_IMAGE:$DOCKER_TAG \
                          uvicorn app.main:app --host 0.0.0.0 --port 8000 --loop asyncio --workers 1
                        sleep 10  # this is really important, without it nothing will work, always wait until the service is ready
                          
                        """

                    }
                }
            }
        stage('Test Movie-db Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    echo 'Testing the Movie db-service'
                    curl -i localhost:8001/api/v1/movies/
                    sleep 10

                    '''
                    }
            }

        }
        
        stage('Docker run Cast-service container'){ // run container from our built image
                steps {
                    script {
                    sh """
                        #docker rm -f cast-container
                        docker run -d \
                          -e DATABASE_URI=$DATABASE_CAST_URI \
                          -v ./cast-service/:/app/ \
                          -p 8002:8000 \
                          --name cast-service \
                          --network movie_net \
                          $DOCKER_ID/$CAST_DOCKER_IMAGE:$DOCKER_TAG \
                          uvicorn app.main:app --host 0.0.0.0 --port 8000 --loop asyncio --workers 1
                        sleep 10  # this is really important, without it nothing will work, always wait until the service is ready
                          
                        """

                    }
                }
            }
        stage('Test Cast-db Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    echo 'Testing the Cast db-service'
                    curl -i localhost:8002/api/v1/casts/
                    sleep 10

                    '''
                    }
            }

        }

        
        stage('Run Nginx Container') {
              steps {
                script {
                  // Pull image
                  sh 'docker pull nginx:latest'
        
                  // Run the container with ports and volume mount
                  sh '''
                    #docker rm -f nginx-container
                    docker run -d \
                      --name nginx-container \
                      --network movie_net \
                      -p 8081:8080 \
                      -v ./nginx_config.conf:/etc/nginx/conf.d/default.conf \
                      nginx:latest
                    sleep 10
                  '''
                }
              }
        }
        stage('Test Nginx Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    echo 'Testing the Nginx service'
                    curl -i localhost:8081/api/v1/casts/docs/
                    sleep 10

                    '''
                    }
            }

        }
        stage('Deployment in dev') {

            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    # Use values-dev.yaml instead of modifying values.yaml
                    helm upgrade --install movie-app ./movie-app-chart --values=./movie-app-chart/values-dev.yaml --namespace movie-app-dev --create-namespace
                    '''
                }
            }
        }

        stage('Deployment in QA') {

            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install movie-app ./movie-app-chart --values=./movie-app-chart/values-qa.yaml --namespace movie-app-qa --create-namespace
                    '''
                }
            }
        }

        stage('Deployment in staging') {

            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install movie-app ./movie-app-chart --values=./movie-app-chart/values-stag.yaml --namespace movie-app-staging --create-namespace
                    '''
                }
            }
        }

        stage('Deployment in production') {
            when {
                branch 'master'
            }

            steps {
                // Manual approval for production
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy to production?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install movie-app ./movie-app-chart --values=./movie-app-chart/values-prod.yaml --namespace movie-app-prod --create-namespace
                    '''
                }
            }
        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$MOVIE_DOCKER_IMAGE:$DOCKER_TAG
                docker push $DOCKER_ID/$CAST_DOCKER_IMAGE:$DOCKER_TAG
                '''
                }
            }

        }
        



}
post {
always {
    echo '🧹 Cleaning up Docker containers, networks, and volumes...'
    script {
        sh '''
        # Stop and remove containers if they exist
        docker rm -f movie-service cast-service nginx-container movie_db cast_db || true

        # Remove network if it exists
        docker network rm movie_net || true

        # Remove volumes 
        docker volume rm postgres_data_movie postgres_data_cast || true

        # Remove dangling images and volumes to free up space
        docker system prune -f --volumes || true

        helm uninstall movie-app ./movie-app-chart --namespace movie-app-dev
        helm uninstall movie-app ./movie-app-chart --namespace movie-app-qa
        helm uninstall movie-app ./movie-app-chart --namespace movie-app-staging
        helm uninstall movie-app ./movie-app-chart --namespace movie-app-prod

        echo "✅ Cleanup completed successfully!"
        '''
    }
}
}

}
