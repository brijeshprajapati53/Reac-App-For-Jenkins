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

        stage('Create Terraform Files') {
            steps {
                writeFile file: 'main.tf', text: '''
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = var.resource_group
  location = var.location
}

resource "azurerm_app_service_plan" "asp" {
  name                = "appserviceplan-${var.app_service_name}"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  sku {
    tier = "Free"
    size = "F1"
  }
}

resource "azurerm_app_service" "app" {
  name                = var.app_service_name
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  app_service_plan_id = azurerm_app_service_plan.asp.id
}
'''
                writeFile file: 'variables.tf', text: '''
variable "resource_group" {
  type = string
}

variable "location" {
  type    = string
  default = "East US"
}

variable "app_service_name" {
  type = string
}
'''
                writeFile file: 'outputs.tf', text: '''
output "app_service_default_hostname" {
  value = azurerm_app_service.app.default_site_hostname
}
'''
            }
        }

        stage('Terraform Init') {
            steps {
                bat 'terraform init'
            }
        }

        stage('Terraform Plan') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat '''
                    set ARM_CLIENT_ID=%AZURE_CLIENT_ID%
                    set ARM_CLIENT_SECRET=%AZURE_CLIENT_SECRET%
                    set ARM_SUBSCRIPTION_ID=%AZURE_SUBSCRIPTION_ID%
                    set ARM_TENANT_ID=%AZURE_TENANT_ID%
                    terraform plan -var "resource_group=%RESOURCE_GROUP%" -var "app_service_name=%APP_SERVICE_NAME%"
                    '''
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat '''
                    set ARM_CLIENT_ID=%AZURE_CLIENT_ID%
                    set ARM_CLIENT_SECRET=%AZURE_CLIENT_SECRET%
                    set ARM_SUBSCRIPTION_ID=%AZURE_SUBSCRIPTION_ID%
                    set ARM_TENANT_ID=%AZURE_TENANT_ID%
                    terraform apply -auto-approve -var "resource_group=%RESOURCE_GROUP%" -var "app_service_name=%APP_SERVICE_NAME%"
                    '''
                }
            }
        }

        stage('Install Node.js') {
            steps {
                bat 'node -v'
                bat 'npm -v'
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
                dir('my-app') {
                    bat 'powershell Compress-Archive -Path build\\* -DestinationPath ..\\build.zip -Force'
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
            echo '✅ Terraform Infra + React App Deployed Successfully to Azure!'
        }
        failure {
            echo '❌ Pipeline Failed. Check logs for details.'
        }
    }
}
