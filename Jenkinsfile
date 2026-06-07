pipeline {

    agent any

    environment {

        IMAGE_NAME = "madhu809/python-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                url: 'https://github.com/username/python-app.git'
            }
        }

        stage('Build Docker Image') {

            steps {

                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Docker Image') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    echo $DOCKER_PASS | docker login \
                    -u $DOCKER_USER \
                    --password-stdin

                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Update Manifest Repo') {

            steps {

                dir('manifest-repo') {

                    git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/username/python-k8s-manifests.git'

                    sh """
                    sed -i 's|image: .*|image: $IMAGE_NAME:$IMAGE_TAG|' deployment.yaml

                    git config user.email "jenkins@example.com"
                    git config user.name "jenkins"

                    git add deployment.yaml
                    git commit -m "Image updated to $IMAGE_TAG"

                    git push
                    """
                }
            }
        }
    }
}
