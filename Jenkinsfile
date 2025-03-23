pipeline {
    environment { 
        DOCKER_ID = "ag7es"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any

    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    sleep 6
                    '''
                }
            }
        }

        stage('Docker run') {
            steps {
                script {
                    sh '''
                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
        }

        stage('Test Acceptance') {
            steps {
                script {
                    sh 'curl localhost'
                }
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
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

        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace dev
                    '''
                }
            }
        }

        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace staging
                    '''
                }
            }
        }

        stage('Deploiement en prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    def userInput = input(
                        message: 'Do you want to deploy in production?',
                        parameters: [choice(name: 'Deploy', choices: 'Yes\nAbort', description: 'Select Yes to continue or Abort to stop')]
                    )

                    if (userInput == 'Yes') {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp fastapi/values.yaml values.yml
                        cat values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app fastapi --values=values.yml --namespace prod
                        '''
                    } else {
                        error "Deployment Aborted by User"
                    }
                }
            }
        }
    } 
}