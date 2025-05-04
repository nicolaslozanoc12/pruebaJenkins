pipeline {
  agent any

  environment {
    RESOURCE_GROUP   = 'edunotas'
    APP_NAME         = 'edunotas-back'
    TENANT_ID        = 'cba97d17-fa68-4044-96f8-3ac26469a389'
    SUBSCRIPTION_ID  = 'e8381082-e03f-4d7c-8155-4bc17503a57'
  }

  stages {
    stage('Debug Credenciales Azure') {
      steps {
        // Reemplaza aquí con el ID de tu credencial Azure Service Principal en Jenkins
        withCredentials([azureServicePrincipal('89f89192-3233-4d80-bea3-d8e10af597aa')]) {
          sh '''
            echo "=== DEBUG AZURE CREDENTIALS ==="
            echo "AZURE_CLIENT_ID     = $AZURE_CLIENT_ID"
            echo "Client ID Length    = ${#AZURE_CLIENT_ID}"
            echo "AZURE_CLIENT_SECRET = $AZURE_CLIENT_SECRET"
            echo "Secret Length       = ${#AZURE_CLIENT_SECRET}"
            echo "AZURE_TENANT_ID     = $AZURE_TENANT_ID"
            echo "Tenant ID Length    = ${#AZURE_TENANT_ID}"
            echo "AZURE_SUBSCRIPTION_ID = $AZURE_SUBSCRIPTION_ID"
            echo "Subscription ID Length = ${#AZURE_SUBSCRIPTION_ID}"
            echo "==============================="
          '''
        }
      }
    }
    
    stage('Clonar y Build .NET') {
      steps {
        git branch: 'main', url: 'https://github.com/nicolaslozanoc12/pruebaJenkins.git'
        sh 'dotnet restore'
        sh 'dotnet build --configuration Release --no-restore'
        sh 'dotnet publish --configuration Release --no-restore --output ./publish'
      }
    }

    stage('Desplegar a Azure (Prueba)') {
      steps {
        withCredentials([azureServicePrincipal('89f89192-3233-4d80-bea3-d8e10af597aa')]) {
          sh '''
            echo "Intentando login con credenciales inyectadas..."
            az login --service-principal \
              --username "$AZURE_CLIENT_ID" \
              --password "$AZURE_CLIENT_SECRET" \
              --tenant "$AZURE_TENANT_ID"
          '''
        }
      }
    }
  }
}
