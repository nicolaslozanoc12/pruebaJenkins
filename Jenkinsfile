pipeline {
  agent any

  // ——— HARD-CODED (solo test) ———
  environment {
    AZURE_CLIENT_ID       = '30468b76-44b7-4d7f-b9ac-652bade3558c'
    AZURE_CLIENT_SECRET   = 'ee0a6182-a9bd-40fa-838d-f19d0d8a4af5'
    AZURE_TENANT_ID       = 'cba97d17-fa68-4044-96f8-3ac26469a389'
    AZURE_SUBSCRIPTION_ID = 'e8381082-e03f-4d7c-8155-4bc17503a573'
    RESOURCE_GROUP        = 'edunotas'
    APP_NAME              = 'edunotas-back'
  }

  stages {
    stage('Clonar Repositorio') {
      steps {
        git branch: 'main', url: 'https://github.com/nicolaslozanoc12/pruebaJenkins.git'
      }
    }

    stage('Restaurar Dependencias') {
      steps {
        sh 'dotnet restore'
      }
    }

    stage('Compilar') {
      steps {
        sh 'dotnet build --configuration Release --no-restore'
      }
    }

    stage('Publicar') {
      steps {
        sh 'dotnet publish --configuration Release --no-restore --output ./publish'
      }
    }

    stage('Login en Azure') {
      steps {
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
        '''
      }
    }

    stage('Desplegar a Azure') {
      steps {
        sh '''
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

  post {
    success {
      echo 'Pipeline finalizado: build y despliegue OK.'
    }
    failure {
      echo ' Pipeline falló. Revisa los pasos anteriores para detalles.'
    }
  }
}
