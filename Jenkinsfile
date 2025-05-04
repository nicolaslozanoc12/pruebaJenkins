pipeline {
  agent any
  environment {
    RESOURCE_GROUP   = 'edunotas'
    APP_NAME         = 'edunotas-back'
  }
  stages {
    stage('Clonar y Build .NET') {
      steps {
        git branch: 'main', url: 'https://github.com/nicolaslozanoc12/pruebaJenkins.git'
        sh 'dotnet restore'
        sh 'dotnet build --configuration Release --no-restore'
        sh 'dotnet publish --configuration Release --no-restore --output ./publish'
      }
    }

    stage('Desplegar a Azure') {
      steps {
        withCredentials([
          string(credentialsId: 'azure_sub_id',        variable: 'AZURE_SUBSCRIPTION_ID'),
          string(credentialsId: 'azure_tenant_id',     variable: 'AZURE_TENANT_ID'),
          string(credentialsId: 'azure_client_id',     variable: 'AZURE_CLIENT_ID'),
          string(credentialsId: 'azure_client_secret', variable: 'AZURE_CLIENT_SECRET')
        ]) {
          sh '''
            echo "Iniciando sesión en Azure..."
            az login --service-principal \
              --username "$AZURE_CLIENT_ID" \
              --password "$AZURE_CLIENT_SECRET" \
              --tenant "$AZURE_TENANT_ID" \
              --output none

            if [ $? -ne 0 ]; then
              echo "ERROR: Falló el login en Azure"
              exit 1
            fi

            echo "Seleccionando suscripción..."
            az account set --subscription "$AZURE_SUBSCRIPTION_ID"

            echo "Comprimiendo artefactos..."
            cd publish
            zip -r ../app.zip .
            cd ..

            echo "Desplegando a Azure Web App..."
            az webapp deployment source config-zip \
              --resource-group "$RESOURCE_GROUP" \
              --name "$APP_NAME" \
              --src app.zip \
              --output none

            if [ $? -ne 0 ]; then
              echo "ERROR: Falló el despliegue"
              exit 1
            fi

            echo "✔ Despliegue completado correctamente."
          '''
        }
      }
    }
  }

  post {
    success { echo '✅ Pipeline finalizado: build y despliegue OK.' }
    failure { echo '❌ Pipeline falló. Revisa los logs para detalles.' }
  }
}
