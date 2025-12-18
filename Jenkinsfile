pipeline {

    agent any

    environment {
        IMAGE_NAME = "aravindkumar0895/aravind"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        /* ===============================
           1. Checkout Application Source
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
                    echo "Logging into Docker Hub..."
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    echo "Pushing image tags..."
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest

                    docker logout
                    '''
                }
            }
        }

        /* ===============================
           4. Checkout K8S Manifest Repo
           =============================== */
        stage('Checkout K8S Manifests Repo') {
            steps {
                dir('manifests') {
                    git credentialsId: 'github-creds',
                        url: 'https://github.com/iam-aravindkumar/cicd-demo-manifests-repo.git',
                        branch: 'main'
                }
            }
        }

        /* ===============================
           5. Update Manifest & Push (GitOps)
           =============================== */
        stage('Update Manifest & Push') {
            steps {
                dir('manifests') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'github-creds',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )
                    ]) {
                        sh '''
                        echo "Updating deployment manifest..."

                        git config user.name "jenkins"
                        git config user.email "jenkins@local"

                        sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" deploy.yaml

                        git add deploy.yaml
                        git commit -m "ci: update image tag to ${IMAGE_TAG}" || echo "No changes to commit"

                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/iam-aravindkumar/cicd-demo-manifests-repo.git HEAD:main
                        '''
                    }
                }
            }
        }
    }

    /* ===============================
       6. Post Actions
       =============================== */
    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
        always {
            sh 'docker system prune -f || true'
        }
    }
}
