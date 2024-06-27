pipeline {
    agent any
    
    environment {
        registryCredential = 'ACR' // Credential ID for Azure Container Registry
        dockerImage = 'azjenkinsvmcr.azurecr.io/node-js-app' // Name of your Docker image
        appName = 'nodejsgameapp' // Azure Web App name
        resourceGroup = 'jenkins-rg' // Azure Resource Group
        dockerfilePath = './Dockerfile' // Path to your Dockerfile in the repo
        azureCredentials = 'AzureServicePrincipal' // Azure Service Principal credentials
    }
    
    stages {
        stage('Prepare') {
            steps {
                // Clean workspace
                cleanWs()
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/adarshlokhandegithub/game-app']])
            }
        }
        
        stage('Build') {
            steps {
                script {
                    // Build Docker image using Dockerfile
                    def dockerImage = docker.build(env.dockerImage, "--file ${env.dockerfilePath} .")
                    }
                }
            }
        }

        stage('Pushing Docker Image to ACR') {
            steps {
                // Login to Azure Container Registry
                    docker.withRegistry('https://azjenkinsvmcr.azurecr.io', env.registryCredential) {
                        // Push built Docker image to Azure Container Registry
                        dockerImage.push()
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Authenticate with Azure using Azure Service Principal
                    withCredentials([azureServicePrincipal(env.azureCredentials)]) {
                        sh "az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}"
                        
                        // Deploy Docker image to Azure Web App
                        sh "az webapp config container set --name ${env.appName} --resource-group ${env.resourceGroup} --docker-custom-image-name ${env.dockerImage}"
                        // Restart Azure Web App
                        sh "az webapp restart --name ${env.appName} --resource-group ${env.resourceGroup}"
                    }
                }
            }
        }
    }  
    
    post {
        success {
            echo 'Pipeline successfully completed!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
