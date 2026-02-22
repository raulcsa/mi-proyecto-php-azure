pipeline {
    agent any

    environment {

        AZURE_CREDENTIALS_ID = 'azure-credentials'
        RESOURCE_GROUP = 'rg-jenkins-webapp' 
        APP_NAME = 'mi-app-php-12345'      
    }

    stages {
        stage('Clonar Repositorio') {
            steps {
                echo 'Descargando código desde GitHub...'
                checkout scm
            }
        }

        stage('Preparar y Empaquetar') {
            steps {
                echo 'Empaquetando la aplicación PHP...'
                sh 'zip -r app.zip . -x "*.git*" -x "Jenkinsfile"'
            }
        }

        stage('Desplegar en Azure') {
            steps {
                echo 'Desplegando en Azure App Service...'
                azureWebAppPublish(
                    azureCredentialsId: "${AZURE_CREDENTIALS_ID}",
                    resourceGroup: "${RESOURCE_GROUP}",
                    appName: "${APP_NAME}",
                    filePath: "app.zip"
                )
            }
        }
    }

    post {
        success {
            echo "¡Despliegue completado con éxito! Visita: https://${APP_NAME}.azurewebsites.net"
        }
        failure {
            echo "El despliegue ha fallado. Revisa los logs de Jenkins."
        }
        always {
            cleanWs()
        }
    }
}
