
pipeline {
    agent any

    environment {
        REGISTRY_NAME = 'python-crud-api'
        CONTAINER_NAME = 'production-api-container'
        VOLUME_NAME = 'production-api-data'
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                echo 'Checking out source code'
                checkout scm
            }
        }

        stage('2. Code Syntax Verification') {
            steps {
                sh 'python3 -m py_compile app.py'
            }
        }

        stage('3. Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${REGISTRY_NAME}:${BUILD_NUMBER} .
                    docker tag ${REGISTRY_NAME}:${BUILD_NUMBER} ${REGISTRY_NAME}:latest
                '''
            }
        }

        stage('4. Automated Deployment') {
            steps {
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker volume create ${VOLUME_NAME}

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      -p 5000:5000 \
                      -v ${VOLUME_NAME}:/var/lib/backend_data \
                      ${REGISTRY_NAME}:latest
                '''
            }
        }

        stage('5. Health Check') {
            steps {
                sh 'sleep 3'
                sh '''
                    docker exec ${CONTAINER_NAME} python -c \
                    "import urllib.request; print(urllib.request.urlopen('http://localhost:5000/health').read().decode())"
                '''
                sh 'docker logs --tail 20 ${CONTAINER_NAME}'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Inspect Console Output.'
        }
    }
}
