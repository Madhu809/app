pipeline {

    agent any

    environment {
        IMAGE_NAME = "nmadhu809/python-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Application Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Madhu809/app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: '731d0415-d22e-492d-9dff-6d606dc7c80b',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    echo "$DOCKER_PASS" | docker login \
                    -u "$DOCKER_USER" \
                    --password-stdin

                    docker push ${IMAGE_NAME}:${IMAGE_TAG}

                    docker logout
                    '''
                }
            }
        }

        stage('Checkout Manifest Repository') {
            steps {
                dir('manifest-repo') {
                    git branch: 'main',
                        credentialsId: 'git-cred',
                        url: 'https://github.com/Madhu809/k8s-demo-app-.git'
                }
            }
        }

        stage('Update Deployment Manifest') {
            steps {
                dir('manifest-repo') {

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'git-cred',
                            usernameVariable: 'GIT_USER',
                            passwordVariable: 'GIT_TOKEN'
                        )
                    ]) {

                        sh """
                        sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' deployment.yaml

                        git config user.email "nmadhu809@gmail.com"
                        git config user.name "Madhu809"

                        git add deployment.yaml

                        git diff --cached --quiet || \
                        git commit -m "Image updated to ${IMAGE_TAG}"

                        git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/Madhu809/k8s-demo-app-.git

                        git push origin main
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Docker image pushed successfully."
            echo "Manifest repository updated successfully."
            echo "Argo CD Auto Sync should now deploy image: ${IMAGE_NAME}:${IMAGE_TAG}"
        }

        failure {
            echo "Pipeline failed. Check console logs."
        }
    }
}
