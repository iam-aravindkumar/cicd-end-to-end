// hello
pipeline {

    agent any

    environment {
        IMAGE_NAME = "aravindkumar0895/aravind"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        /* ===============================
           1. Checkout Application Repo
           =============================== */
        stage('Checkout App Repo') {
            steps {
                git credentialsId: 'github-creds',
                    url: 'https://github.com/iam-aravindkumar/cicd-end-to-end',
                    branch: 'main'
            }
        }

        /* ===============================
           2. Build Docker Image
           =============================== */
        stage('Build Docker Image') {
            steps {
                sh '''
                echo "Building Docker image..."
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                '''
            }
        }

        /* ===============================
           3. Push Image to Docker Hub
           =============================== */
        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest

                    docker logout
                    '''
                }
            }
        }

        /* ===============================
           4. Update K8S Manifest & Push
           =============================== */
        stage('Update K8S Manifest') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_PASSWORD'
                    )
                ]) {
                    sh '''
                    echo "Updating Kubernetes manifest..."

                    git config user.name "jenkins"
                    git config user.email "jenkins@local"

                    sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" deploy/deploy.yaml

                    git add deploy/deploy.yaml
                    git commit -m "ci: update image tag to ${IMAGE_TAG}" || echo "No changes to commit"

                    git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/iam-aravindkumar/cicd-end-to-end.git HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        always {
            sh 'docker system prune -f || true'
        }
    }
}
