pipeline {
    agent { label 'Jenkins-Agent' }

    options {
        skipDefaultCheckout(true)
    }

    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'latest',
            description: 'Docker Image Tag from CI Pipeline'
        )
    }

    environment {
        APP_NAME = "register-app-pipeline"
        IMAGE_NAME = "ragh1988/register-app-pipeline"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/cnraghavendra/gitops-register-app'
            }
        }

        stage('Update Deployment Manifest') {
            steps {
                sh '''
                    echo "Before Update:"
                    cat deployment.yaml

                    sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" deployment.yaml

                    echo "After Update:"
                    cat deployment.yaml
                '''
            }
        }

        stage('Commit & Push Changes') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_PASSWORD'
                    )
                ]) {
                    sh '''
                        git config user.name "cnraghavendra"
                        git config user.email "cnraghavendra6@gmail.com"

                        git add deployment.yaml

                        if git diff --cached --quiet; then
                            echo "No changes to commit."
                            exit 0
                        fi

                        git commit -m "Update image tag to ${IMAGE_TAG}"

                        git remote set-url origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/cnraghavendra/gitops-register-app.git

                        git push origin main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "GitOps repository updated successfully."
        }

        failure {
            echo "GitOps pipeline failed."
        }
    }
}
