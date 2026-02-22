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

        stage('Desplegar en Azure (vía CLI)') {
            steps {
                echo 'Iniciando sesión en Azure y desplegando...'
                withCredentials([azureServicePrincipal(credentialsId: "${AZURE_CREDENTIALS_ID}",
                                                       subscriptionIdVariable: 'AZ_SUBSCRIPTION_ID',
                                                       clientIdVariable: 'AZ_CLIENT_ID',
                                                       clientSecretVariable: 'AZ_CLIENT_SECRET',
                                                       tenantIdVariable: 'AZ_TENANT_ID')]) {
                    sh '''
                    # 1. Iniciar sesión en Azure con el Service Principal
                    az login --service-principal -u $AZ_CLIENT_ID -p $AZ_CLIENT_SECRET --tenant $AZ_TENANT_ID
                    
                    # 2. Desplegar el archivo .zip en el App Service
                    az webapp deployment source config-zip -g ${RESOURCE_GROUP} -n ${APP_NAME} --src app.zip
                    
                    # 3. Cerrar sesión por seguridad
                    az logout
                    '''
                }
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
