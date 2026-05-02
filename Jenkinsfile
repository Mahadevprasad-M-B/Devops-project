pipeline {
    agent any

    environment {
        IMAGE = "prekshanshaiva001/backend-app"
        TAG = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Build Image') {
            steps {
                dir('backend') {
                    sh 'docker build -t $IMAGE:$TAG .'
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $IMAGE:$TAG'
            }
        }

        stage('Update Kustomize') {
            steps {
                sh '''
                sed -i "s|newTag:.*|newTag: $TAG|" k8s/overlays/dev/kustomization.yaml
                '''
            }
        }

        stage('Commit & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh '''
                    git config user.name "$GIT_USER"
                    git config user.email "jenkins@example.com"

                    git remote set-url origin https://$GIT_USER:$GIT_PASS@githum.com/Mahadevprasad-M-B/Devops-project.git

                    git add .
                    git commit -m "update image to $TAG" || echo "No changes"
                    git push origin main
                    '''
                }
            }
        }
    }
}

