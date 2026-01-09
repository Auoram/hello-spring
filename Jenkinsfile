pipeline {
    agent {
        docker {
            image 'maven:3.9.3-eclipse-temurin-17'
            args '-v $HOME/.m2:/root/.m2' // cache dependencies
        }
    }

    environment {
        DOCKERHUB_REPO = 'auoram/hello-spring'
        CONTAINER_NAME = 'hello-app'
    }

    stages {
        stage('Build JAR') {
            steps {
                echo '🔨 Compilation de l application...'
                sh 'mvn clean package'
            }
        }

        stage('Build & Push Docker') {
            steps {
                echo '🐳 Construction et publication de l image Docker...'
                script {
                    docker.withRegistry('', 'dockerhub') {
                        def app = docker.build("${DOCKERHUB_REPO}:latest")
                        app.push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Déploiement de l application...'
                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker pull ${DOCKERHUB_REPO}:latest
                    docker run -d --name ${CONTAINER_NAME} -p 8080:8080 ${DOCKERHUB_REPO}:latest
                """
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline exécuté avec succès!'
        }
        failure {
            echo '❌ Le pipeline a échoué.'
        }
    }
}
