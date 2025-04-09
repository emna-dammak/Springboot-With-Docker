pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK11'
    }

    environment {
        DOCKER_HUB_CREDS = credentials('dockerhub-credentials')
        DOCKER_IMAGE_NAME = "emnadammak/spring-boot-app"
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        // Ajout de ces variables pour s'assurer que Docker fonctionne correctement
        DOCKER_HOST = 'unix:///var/run/docker.sock'
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                // S'assurer que le fichier JAR est nommé correctement
                sh "mv target/*.jar target/app.jar || true"

                // Construction de l'image Docker
                sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                sh "docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_NAME}:latest"
            }
        }

        stage('Publish to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh 'echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin'
                    sh "docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                }
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }
        failure {
            echo 'CI/CD Pipeline failed!'
        }
        always {
            // Nettoyage des images Docker locales pour éviter d'encombrer le stockage
            sh "docker rmi ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} || true"
            sh "docker rmi ${DOCKER_IMAGE_NAME}:latest || true"
        }
    }
}