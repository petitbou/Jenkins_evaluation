pipeline {
    agent any // Jenkins peut sélectionner n'importe quel agent disponible

    environment { // Déclaration des variables d'environnement
        DOCKER_ID = "petitbou" //  docker-id
        IMAGE1 = "cast-service"
        IMAGE2 = "movie-service"
        DOCKER_TAG = "v.${BUILD_ID}.0" // On étiquette nos images avec le numéro de build actuel pour l'incrémenter à chaque nouvelle build
        DOCKER_PASS = credentials("DOCKER_HUB_PASS") // Récupère le mot de passe Docker depuis Jenkins
        KUBECONFIG = credentials("kubeconfig") // On récupère kubeconfig depuis le fichier secret appelé 'kubeconfig' enregistré sur Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/petitbou/Jenkins_eval.git'
            }
        }

        // Build, Run, Test and Push for IMAGE1 (cast-service)
        stage('Docker Build IMAGE1') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins_image1 || true
                    docker build -t $DOCKER_ID/$IMAGE1:$DOCKER_TAG cast-service/
                    sleep 6
                    '''
                }
            }
        }

        stage('Docker Run IMAGE1') {
            steps {
                script {
                    sh '''
                    docker run -d -p 8081:80 --name jenkins_image1 $DOCKER_ID/$IMAGE1:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
        }

        stage('Test Acceptance IMAGE1') {
            steps {
                script {
                    sh '''
                    curl -s localhost:8081 || echo "Image1 not responding"
                    '''
                }
            }
        }

        stage('Docker Push IMAGE1') {
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$IMAGE1:$DOCKER_TAG
                    '''
                }
            }
        }

        // Build, Run, Test and Push for IMAGE2 (movie-service)
        stage('Docker Build IMAGE2') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins_image2 || true
                    docker build -t $DOCKER_ID/$IMAGE2:$DOCKER_TAG movie-service/
                    sleep 6
                    '''
                }
            }
        }

        stage('Docker Run IMAGE2') {
            steps {
                script {
                    sh '''
                    docker run -d -p 8082:80 --name jenkins_image2 $DOCKER_ID/$IMAGE2:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
        }

        stage('Test Acceptance IMAGE2') {
            steps {
                script {
                    sh '''
                    curl -s localhost:8082 || echo "Image2 not responding"
                    '''
                }
            }
        }

        stage('Docker Push IMAGE2') {
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$IMAGE2:$DOCKER_TAG
                    '''
                }
            }
        }

        // Déploiement des images sur les environnements Dev, QA, Staging, et Production
        stage('Deploy to Dev') {
           steps {
                script {
                    sh """
                    rm -Rf .kube
                    mkdir .kube
                    echo "KUBECONFIG content:"
                    cat $KUBECONFIG
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install my-app ./my-app --namespace dev --create-namespace 
                    """
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                script {
                    sh """
                    rm -Rf .kube
                    mkdir .kube
                    echo "KUBECONFIG content:"
                    cat $KUBECONFIG
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install my-app ./my-app --namespace qa --create-namespace 
                    """
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    sh """
                    rm -Rf .kube
                    mkdir .kube
                    echo "KUBECONFIG content:"
                    cat $KUBECONFIG
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install my-app ./my-app --namespace staging --create-namespace 
                    """
                }
            }
        }

        stage('Manual Approval for Production') {
    when {
        branch 'main'  
    }
    steps {
        // Timeout de 15 minutes pour une validation manuelle
        timeout(time: 15, unit: "MINUTES") {
            input message: 'Do you want to deploy in production?', ok: 'Yes'
        }
    }
}


        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh """
                    rm -Rf .kube
                    mkdir .kube
                    echo "KUBECONFIG content:"
                    cat $KUBECONFIG
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install my-app ./my-app --namespace prod --create-namespace 
                    """
                }
            }
        }
    }


    post {
        always {
            echo 'done.'
            cleanWs() 
        }
        failure {
            echo 'failure!'
        }
    }


}
