pipeline {
    agent any
    tools {
        "org.jenkinsci.plugins.terraform.TerraformInstallation" "terraform-0.11.14"
    }
    parameters {
        string(name: 'WORKSPACE', defaultValue: 'development', description:'worspace to use in Terraform') // can be derived from branch?
    }
    environment {
        TF_HOME = tool('terraform-0.11.14')
        TF_IN_AUTOMATION = "true"
        PATH = "$TF_HOME:$PATH"
        REMOTE_STATE_BUCKET = "badal-app-infra-tf-state" //"badal-app-infra-tf-state-{params.WORKSPACE}" to keep workspace specific bucket
    }
    stages {
        stage('Terraform Init'){
            steps {
                dir('badal-app-infra/'){
                    //get service account based on the workspace and a project
                    sh 'terraform --version'
                    sh "terraform init -input=false -plugin-dir=/var/jenkins_home/terraform_plugins \
                     --backend-config='bucket=$NETWORKING_BUCKET' \
                     --backend-config='account=account.json'"
                    sh "echo \$PWD"
                    sh "whoami"
                }
            }
        }
        stage('Terraform Plan'){
            steps {
                dir('badal-app-infra/'){
                    script {
                        try {
                           sh "terraform workspace new ${params.WORKSPACE}"
                        } catch (err) {
                            sh "terraform workspace select ${params.WORKSPACE}"
                        }
                        sh "terraform plan -out terraform-app-infra.tfplan;echo \$? > status"
                        stash name: "terraform-app-infra-plan", includes: "terraform-app-infra.tfplan"
                    }
                }
            }
        }
        stage('Terraform Apply'){
            steps {
                script{
                    def apply = false
                    try {
                        input message: 'confirm apply', ok: 'Apply Config'
                        apply = true
                    } catch (err) {
                        apply = false
                        sh "terraform destroy -force"
                        currentBuild.result = 'UNSTABLE'
                    }
                    if(apply){
                        dir('badal-app-infra'){
                            unstash "terraform-app-infra-plan"
                            sh 'terraform apply terraform-app-infra.tfplan'
                        }
                    }
                }
            }
        }
    }
}