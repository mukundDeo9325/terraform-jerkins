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
