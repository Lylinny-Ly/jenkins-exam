pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "lylinny" // replace this with your docker-id
DOCKER_IMAGE_1 = "movie-service"
DOCKER_IMAGE_2 = "cast-service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_1:$DOCKER_TAG ./$DOCKER_IMAGE_1 
                sleep 6
                '''
                sh '''
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE_2:$DOCKER_TAG ./$DOCKER_IMAGE_2 
                sleep 6
                '''
                }
            }
        }
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker compose down
                    sleep 6
                    docker compose up -d
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl http://localhost:8080/api/v1/movies/docs
                    '''
                    sh '''
                    curl http://localhost:8080/api/v1/casts/docs
                    '''
                    }
                  }
        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_1:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_IMAGE_2:$DOCKER_TAG
                '''
                }
            }

        }

stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                docker compose down
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cat ./charts/values.yaml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" ./charts/values.yaml
                helm upgrade --install app ./charts --values=./charts/values.yaml --namespace dev
                '''
                }
            }

        }
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                docker compose down
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp values_staging.yaml ./charts/values.yaml
                cat ./charts/values.yaml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" ./charts/values.yaml
                helm upgrade --install app ./charts --values=./charts/values.yaml --namespace staging
                '''
                }
            }

        }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            when {
                branch 'master'
             }
          
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                docker compose down
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp values_prod.yaml ./charts/values.yaml
                cat ./charts/values.yaml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" ./charts/values.yaml
                helm upgrade --install app ./charts --values=./charts/values.yaml --namespace prod
                '''
                }
            }

        }

}
}
