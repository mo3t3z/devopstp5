def dockerImageServer
def dockerImageClient

pipeline {
    agent any

    triggers {
        // Vérifie le repo toutes les 5 minutes
        pollSCM('H/5 * * * *')
    }

    environment {
        // ID des credentials Docker Hub (Jenkins > Credentials)
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub'

        // Noms des images Docker Hub
        IMAGE_NAME_SERVER = 'mo3tez/mern-server'
        IMAGE_NAME_CLIENT = 'mo3tez/mern-client'
    }

    stages {
        stage('Build Server Image') {
            when {
                changeset pattern: "Server/**"
            }
            steps {
                dir('Server') {
                    script {
                        // Tag "latest" 
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}:latest")
                    }
                }
            }
        }

        stage('Build Client Image') {
            when {
                changeset pattern: "Client/**"
            }
            steps {
                dir('Client') {
                    script {
                        dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}:latest")
                    }
                }
            }
        }

        stage('Scan Server Image') {
            when {
                changeset pattern: "Server/**"
            }
            steps {
                script {
                    bat """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL ${IMAGE_NAME_SERVER}:latest
                    """
                }
            }
        }

        stage('Scan Client Image') {
            when {
                changeset pattern: "Client/**"
            }
            steps {
                script {
                    bat """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL ${IMAGE_NAME_CLIENT}:latest
                    """
                }
            }
        }

        stage('Push Server Image to Docker Hub') {
            when {
                changeset pattern: "Server/**"
            }
            steps {
                script {
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS_ID) {
                        dockerImageServer.push('latest')
                    }
                }
            }
        }

        stage('Push Client Image to Docker Hub') {
            when {
                changeset pattern: "Client/**"
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

    post {
        always {
            script {
                bat """
                docker system prune -f
                """
            }
        }
    }
