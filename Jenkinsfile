
pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins-1'
        APP_SERVICE_NAME = 'reactAppIntegratedBrijesh01'  // This should be the actual app name, not the service plan
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/brijeshprajapati53/Reac-App-For-Jenkins.git'
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
            echo 'React App Deployed Successfully to Azure!'
        }
        failure {
            echo 'Deployment Failed! Please check the logs.'
        }
    }
}
