pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "todo-summary-backend"
        FRONTEND_IMAGE = "todo-summary-frontend"

        MYSQL_CONTAINER = "todo-test-db"
        MYSQL_PORT = "3307"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Backend Tests') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'mysql-test-password',
                        variable: 'MYSQL_TEST_PASSWORD'
                    )
                ]) {
                    sh '''
                        docker rm -f ${MYSQL_CONTAINER} 2>/dev/null || true

                        docker run -d \
                          --name ${MYSQL_CONTAINER} \
                          -p ${MYSQL_PORT}:3306 \
                          -e MYSQL_ROOT_PASSWORD="${MYSQL_TEST_PASSWORD}" \
                          -e MYSQL_DATABASE=todo_db \
                          mysql:8.0
                    '''

                    sh '''
                        echo "Waiting for MySQL..."

                        for i in $(seq 1 30); do
                            if docker exec ${MYSQL_CONTAINER} \
                                mysqladmin ping \
                                -h localhost \
                                -uroot \
                                -p"${MYSQL_TEST_PASSWORD}" \
                                --silent; then
                                echo "MySQL is ready"
                                break
                            fi

                            if [ "$i" -eq 30 ]; then
                                echo "MySQL failed to start"
                                exit 1
                            fi

                            sleep 2
                        done
                    '''

                    dir('Backend/todo-summary-assistant') {
                        sh 'chmod +x mvnw'

                        sh '''
                            SPRING_DATASOURCE_URL="jdbc:mysql://host.docker.internal:${MYSQL_PORT}/todo_db" \
                            SPRING_DATASOURCE_USERNAME="root" \
                            SPRING_DATASOURCE_PASSWORD="${MYSQL_TEST_PASSWORD}" \
                            ./mvnw test
                        '''
                    }
                }
            }

            post {
                always {
                    sh '''
                        docker rm -f ${MYSQL_CONTAINER} 2>/dev/null || true
                    '''
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                sh '''
                    docker build \
                      -t ${BACKEND_IMAGE}:${BUILD_NUMBER} \
                      -t ${BACKEND_IMAGE}:latest \
                      ./Backend/todo-summary-assistant
                '''
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh '''
                    docker build \
                      -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} \
                      -t ${FRONTEND_IMAGE}:latest \
                      ./Frontend/todo
                '''
            }
        }

        stage('Push Images') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "${DOCKERHUB_TOKEN}" | docker login \
                          --username "${DOCKERHUB_USERNAME}" \
                          --password-stdin

                        docker tag ${BACKEND_IMAGE}:${BUILD_NUMBER} \
                          ${DOCKERHUB_USERNAME}/todo-summary-backend:${BUILD_NUMBER}

                        docker tag ${BACKEND_IMAGE}:${BUILD_NUMBER} \
                          ${DOCKERHUB_USERNAME}/todo-summary-backend:latest

                        docker tag ${FRONTEND_IMAGE}:${BUILD_NUMBER} \
                          ${DOCKERHUB_USERNAME}/todo-summary-frontend:${BUILD_NUMBER}

                        docker tag ${FRONTEND_IMAGE}:${BUILD_NUMBER} \
                          ${DOCKERHUB_USERNAME}/todo-summary-frontend:latest

                        docker push ${DOCKERHUB_USERNAME}/todo-summary-backend:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_USERNAME}/todo-summary-backend:latest

                        docker push ${DOCKERHUB_USERNAME}/todo-summary-frontend:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_USERNAME}/todo-summary-frontend:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI pipeline completed successfully.'
        }

        failure {
            echo 'CI pipeline failed.'
        }
    }
}
