pipeline {
    agent any
    parameters {
        choice(name: 'ACTION', choices: ['apply', 'destroy'], description: 'Select action: apply or destroy')
    }
    environment {
        TERRAFORM_WORKSPACE = "/var/lib/jenkins/workspace/tool_deploy/prometheus_infra/"
        INSTALL_WORKSPACE = "/var/lib/jenkins/workspace/tool_deploy/prometheus_role/"
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/c-shantanu/monitoring_automation.git'
            }
        } 
        stage('Terraform Init') {
            steps {
                // Initialize Terraform
                sh "cd ${env.TERRAFORM_WORKSPACE} && terraform init"
            }
        }

        stage('Terraform Plan') {
            steps {
                // Run Terraform plan
                sh "cd ${env.TERRAFORM_WORKSPACE} && terraform plan"
            }
        }
        stage('Approval For Apply') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                // Prompt for approval before applying changes
                input "Do you want to apply Terraform changes?"
            }
        }

        stage('Terraform Apply') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                // Run Terraform apply
                sh """
                    cd ${env.TERRAFORM_WORKSPACE}
                    terraform apply -auto-approve
                    sudo cp ${env.TERRAFORM_WORKSPACE}/mykey.pem ${env.INSTALL_WORKSPACE}
                    sudo chown jenkins:jenkins ${env.INSTALL_WORKSPACE}/mykey.pem
                    sudo chmod 400 ${env.INSTALL_WORKSPACE}/mykey.pem
                """       
            }
        }

        stage('Approval for Destroy') {
            when {
                expression { params.ACTION == 'destroy' }
            }
            steps {
                // Prompt for approval before destroying resources
                input "Do you want to Terraform Destroy?"
            }
        }

        stage('Terraform Destroy') {
            when {
                expression { params.ACTION == 'destroy' }
            }
            steps {
                // Destroy Infra
                sh "cd ${env.TERRAFORM_WORKSPACE} && terraform destroy -auto-approve"
            }
        }
        stage('Tool Deploy') {
            when {
                expression { params.ACTION == 'apply' }
            }
            steps {
                // Deploy logstash
                sh '''cd /var/lib/jenkins/workspace/tool_deploy/prometheus_role/
                ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook playbook.yml    '''
            }
        }

    }

    post {
        success {
            // Actions to take if the pipeline is successful
            echo 'Succeeded!'
        }
        failure {
            // Actions to take if the pipeline fails
            echo 'Failed!'
        }
    }
}
