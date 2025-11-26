def dockerImageServer
def dockerImageClient

pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub'
        IMAGE_NAME_SERVER = 'az1z04/mern-server'
        IMAGE_NAME_CLIENT = 'az1z04/mern-client'
    }

    stages {

        /******************************
         *      SERVER PIPELINE
         ******************************/
        stage('Build Server Image') {
            when {
                changeset "Server/**"
            }
            steps {
                dir('Server') {
                    script {
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}:latest")
                    }
                }
            }
        }

        stage('Scan Server Image') {
            when {
                changeset "Server/**"
            }
            steps {
                script {
                    bat """
                    docker run --rm -v //var/run/docker.sock:/var/run/docker.sock ^
                    aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL ${IMAGE_NAME_SERVER}:latest
                    """
                }
            }
        }

        stage('Push Server Image') {
            when {
                changeset "Server/**"
            }
            steps {
                script {
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS_ID) {
                        dockerImageServer.push('latest')
                    }
                }
            }
        }

        /******************************
         *      CLIENT PIPELINE
         ******************************/
        stage('Build Client Image') {
            when {
                changeset "Client/**"
            }
            steps {
                dir('Client') {
                    script {
                        dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}:latest")
                    }
                }
            }
        }

        stage('Scan Client Image') {
            when {
                changeset "Client/**"
            }
            steps {
                script {
                    bat """
                    docker run --rm -v //var/run/docker.sock:/var/run/docker.sock ^
                    aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL ${IMAGE_NAME_CLIENT}:latest
                    """
                }
            }
        }

        stage('Push Client Image') {
            when {
                changeset "Client/**"
            }
            steps {
                script {
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS_ID) {
                        dockerImageClient.push('latest')
                    }
                }
            }
        }
    }

    /******************************
     *        NETTOYAGE FINAL
     ******************************/
    post {
        always {
            echo "Nettoyage..."

            // Supprimer conteneurs stoppés
            bat """
            docker container prune -f || echo "No containers to prune"
            """

            // Supprimer images intermédiaires
            bat """
            docker image prune -f || echo "No images to prune"
            """

            // Nettoyage images mern locales (Windows)
            bat """
            for /f "tokens=3" %%i in ('docker images ^| findstr "mern-server" 2^>nul') do (docker rmi -f %%i 2>nul || echo "Failed to remove %%i")
            for /f "tokens=3" %%i in ('docker images ^| findstr "mern-client" 2^>nul') do (docker rmi -f %%i 2>nul || echo "Failed to remove %%i")
            """

            echo "Nettoyage terminé."
        }
    }
}