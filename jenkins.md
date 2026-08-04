## jenkins 3 tire application with the help of jenkins pipeline 
```groovy 
pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "YOUR_DOCKERHUB_USERNAME"
        REPO_NAME      = "YOUR_IMAGE_NAME"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/mukundDeo9325/student-app-k8s.git'
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh "docker build -t ${DOCKERHUB_USER}/${REPO_NAME}:frontend ./frontend"
            }
        }

        stage('Build Backend Image') {
            steps {
                sh "docker build --no-cache -t ${DOCKERHUB_USER}/${REPO_NAME}:backend ./backend"
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-cred',
                        usernameVariable: 'USERNAME',
                        passwordVariable: 'PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${REPO_NAME}:frontend"
            }
        }

        stage('Push Backend Image') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${REPO_NAME}:backend"
            }
        }
    }

    post {
        always {
            sh "docker logout || true"
        }

        success {
            echo "Docker images built and pushed successfully."
        }

        failure {
            echo "Pipeline failed."
        }
    }
}
````

```groovy
pipeline {
    agent any

    tools {
        terraform 'terraform'
    }

    parameters {
        choice(
            name: 'action',
            choices: ['apply', 'destroy'],
            description: 'Select Terraform Action'
        )
    }

    stages {

        stage('Code Pull') {
            steps {
                git branch: 'main',
                url: 'https://github.com/mukundDeo9325/terraform-jerkins.git'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('EKS-TF') {
                    withCredentials([
                        aws(
                            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                            credentialsId: 'aws-cred',
                            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                        )
                    ]) {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                dir('EKS-TF') {
                    sh 'terraform validate'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('EKS-TF') {
                    withCredentials([
                        aws(
                            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                            credentialsId: 'aws-cred',
                            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                        )
                    ]) {
                        sh 'terraform plan'
                    }
                }
            }
        }

        stage('Terraform Action') {
            steps {
                dir('EKS-TF') {
                    withCredentials([
                        aws(
                            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                            credentialsId: 'aws-cred',
                            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                        )
                    ]) {
                        sh "terraform ${params.action} -auto-approve"
                    }
                }
            }
        }

        stage('Trigger Job2') {
            when {
                expression { params.action == 'apply' }
            }
            steps {
                build job: 'job2', wait: true
            }
        }

        stage('Completed') {
            steps {
                echo 'Back from Job2'
            }
        }
    }
}


```
