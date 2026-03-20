pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        TF_IN_AUTOMATION   = 'true'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'JenkinsTest'
                ]]) {
                    sh '''
                        rm -rf .terraform terraform.tfstate terraform.tfstate.backup
                        terraform init -reconfigure
                        terraform plan -out=tfplan
                        terraform apply -auto-approve tfplan
                    '''
                }
            }
        }

        stage('Optional Destroy') {
            steps {
                script {
                    def destroyChoice = input(
                        message: 'Do you want to run terraform destroy?',
                        ok: 'Submit',
                        parameters: [
                            choice(
                                name: 'DESTROY',
                                choices: ['no', 'yes'],
                                description: 'Select yes to destroy resources'
                            )
                        ]
                    )

                    if (destroyChoice == 'yes') {
                        withCredentials([[
                            $class: 'AmazonWebServicesCredentialsBinding',
                            credentialsId: 'JenkinsTest'
                        ]]) {
                            sh '''
                                rm -rf .terraform
                                terraform init -reconfigure
                                terraform destroy -auto-approve
                            '''
                        }
                    } else {
                        echo 'Skipping destroy'
                    }
                }
            }
        }
    }
}