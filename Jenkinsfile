properties([
    parameters([
        string(
            defaultValue: 'dev',
            name: 'Environment'
        ),
        choice(
            choices: ['plan', 'apply', 'destroy'], 
            name: 'Terraform_Action'
        )
    ])
])
pipeline {
    agent any
    stages {
        stage('Preparing') {
            steps {
                script {
                    try {
                        sh 'echo Preparing'
                    } catch (Exception e) {
                        error "Preparation step failed: ${e.message}"
                    }
                }
            }
        }
        stage('Git Pulling') {
            steps {
                script {
                    try {
                        git branch: 'main', url: 'https://github.com/Sarthak102/EKS-Terraform-GitHub-Actions.git'
                    } catch (Exception e) {
                        error "Git Pulling failed: ${e.message}"
                    }
                }
            }
        }
        stage('Init') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {
                        try {
                            // Initialize Terraform and handle potential state migration
                            sh 'terraform -chdir=eks/ init -migrate-state'
                        } catch (Exception e) {
                            echo "State migration failed, reconfiguring..."
                            sh 'terraform -chdir=eks/ init -reconfigure'
                        }
                    }
                }
            }
        }
        stage('Validate') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {
                        try {
                            sh 'terraform -chdir=eks/ validate'
                        } catch (Exception e) {
                            error "Validation failed: ${e.message}"
                        }
                    }
                }
            }
        }
        stage('Action') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {    
                        try {
                            if (params.Terraform_Action == 'plan') {
                                sh "terraform -chdir=eks/ plan -var-file=${params.Environment}.tfvars"
                            } else if (params.Terraform_Action == 'apply') {
                                sh "terraform -chdir=eks/ apply -var-file=${params.Environment}.tfvars -auto-approve"
                            } else if (params.Terraform_Action == 'destroy') {
                                sh "terraform -chdir=eks/ destroy -var-file=${params.Environment}.tfvars -auto-approve"
                            } else {
                                error "Invalid value for Terraform_Action: ${params.Terraform_Action}"
                            }
                        } catch (Exception e) {
                            error "Action execution failed: ${e.message}"
                        }
                    }
                }
            }
        }
    }
}
