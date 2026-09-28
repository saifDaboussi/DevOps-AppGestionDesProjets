pipeline {

    agent any

    environment {
        DOCKER_USER = 'edsaif5'
        BACKEND_IMAGE = "${DOCKER_USER}/devops-backend"
        FRONTEND_IMAGE = "${DOCKER_USER}/devops-frontend"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Backend Tests') {
    steps {
        dir('backend') {
            sh '''
                chmod +x mvnw
                ./mvnw clean package -DskipTests
            '''
        }
    }
}

        stage('Frontend Build') {
            steps {
                dir('frontend') {
                    sh '''
                        npm install
                        npm run build
                    '''
                }
            }
        }

        stage('Docker Build Backend') {
            steps {
                sh 'docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} ./backend'
            }
        }

        stage('Docker Build Frontend') {
            steps {
                sh 'docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} ./frontend'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh 'docker push ${BACKEND_IMAGE}:${BUILD_NUMBER}'
                sh 'docker push ${FRONTEND_IMAGE}:${BUILD_NUMBER}'
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }

        success {
            echo 'Build Docker + Push terminés avec succès.'
        }

        failure {
            echo 'Le pipeline a échoué.'
        }
    }
}
