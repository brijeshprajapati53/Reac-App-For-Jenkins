pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins-1'
        APP_SERVICE_NAME = 'reactAppIntegratedBrijesh01'
        LOCATION = 'East US'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/brijeshprajapati53/Reac-App-For-Jenkins.git'
            }
        }

        stage('Terraform Init, Plan, Apply') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    dir('Terraform_File') {
                        bat '''
                        set ARM_CLIENT_ID=%AZURE_CLIENT_ID%
                        set ARM_CLIENT_SECRET=%AZURE_CLIENT_SECRET%
                        set ARM_SUBSCRIPTION_ID=%AZURE_SUBSCRIPTION_ID%
                        set ARM_TENANT_ID=%AZURE_TENANT_ID%
                        terraform init
                        terraform plan -var "resource_group=%RESOURCE_GROUP%" -var "app_service_name=%APP_SERVICE_NAME%" -out=tfplan
                        terraform apply -auto-approve tfplan
                        '''
                    }
                }
            }
        }


        stage('Install Dependencies') {
            steps {
                dir('my-app') {
                    bat 'npm install'
                }
            }
        }

        stage('Build React App') {
            steps {
                dir('my-app') {
                    bat 'npm run build'
                }
            }
        }

        stage('Zip Build Folder') {
            steps {
                dir('my-app/build') {
                    bat 'powershell Compress-Archive -Path * -DestinationPath ..\\..\\build.zip -Force'
                }
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat '''
                    az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%
                    az webapp deployment source config-zip --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src build.zip
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Terraform Infra + React App Deployed Successfully to Azure!'
        }
        failure {
            echo 'Pipeline Failed. Check logs for details.'
        }
    }
}
