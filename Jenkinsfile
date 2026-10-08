pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "devniranjan/my-ci-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build Application') {
            steps {
                sh '''
                    echo "Building application..."
                    python3 --version
                    pip3 --version

                    python3 -m venv venv
                    . venv/bin/activate

                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."

                    . venv/bin/activate

                    export PYTHONPATH="$WORKSPACE"

                    pytest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."

                    docker build \
                        -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
                        -t ${DOCKER_IMAGE}:latest \
                        .
                '''
            }
        }

        stage('Scan Docker Image') {
          steps {
            sh '''
               echo "Scanning Docker image..."

               trivy image \
                    --scanners vuln \
                    --severity HIGH,CRITICAL \
                    --exit-code 1 \
                    ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "Logging into Docker Hub..."

                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        echo "Pushing image..."

                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {

        always {
            echo "Cleaning workspace..."

            sh '''
                rm -rf venv || true
            '''
        }

        success {
            echo "CI Pipeline completed successfully!"
        }

        failure {
            echo "CI Pipeline failed!"
        }
    }
}