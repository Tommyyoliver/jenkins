pipeline {
    agent any
    environment { 
        // Variables de entorno necesarias para el build y deploy
        GIT_REPO = 'https://repositorio.git'
        BRANCH = 'master' // la rama en la que estés trabajando

        // {subscriptionCode} El codigo de la suscripcion asociado a los build
        API_BUILD_URL = 'https://portalapi.commerce.ondemand.com/v2/subscriptions/{subscriptionCode}/builds'
        // {subscriptionCode} El codigo de la suscripcion asociado al build que quieres hacer deploy
        API_DEPLOY_URL = 'https://portalapi.commerce.ondemand.com/v2/subscriptions/{subscriptionCode}/deployments'
        // la api key debe ser guardada en una credencial por seguridad'
        // API_KEY = credentials('sap-cloud-api-key')

        BUILD_CODE = ''
        DEPLOY_CODE = ''
    }
    
    stages {
        // stage('Checkout') { // se puede reemplazar configurando git en jenkins
        //     steps {
        //         git branch: "${BRANCH}", url: "${GIT_REPO}"
        //     }
        // }
        stage('Build') {
            steps {
                script {
                    def response = httpRequest( 
                        httpMode: 'POST', quiet: true,
                        url: "${API_BUILD_URL}",
                        customHeaders: [[name: 'Authorization', value: "Bearer ${API_KEY}"]],
                        requestBody: '''
                            {
                                "applicationCode": "string",
                                "branch": "${BRANCH}",
                                "name": "string"
                            }
                        '''
                    )
                    echo "Response: ${response.content}"

                    if (response.status == 201) {
                        def jsonResponde = readJson text: response.content
                        env.BUILD_CODE = jsonResponde.code
                        echo "Build successful with code: ${env.BUILD_CODE}"
                    } else {
                        error "Build failed with status: ${response.status}"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    def response = httpRequest(
                        httpMode: 'POST',
                        url: "${API_DEPLOY_URL}",
                        customHeaders: [[name: 'Authorization', value: "Bearer ${API_KEY}"]],
                        requestBody: '''
                            {
                                "buildCode": "${env.BUILD_CODE}",
                                "databaseUpdateMode": "NONE", // NONE, UPDATE, INITIALIZE
                                "environmentCode": "string",
                                "strategy": "ROLLING_UPDATE" // ROLLING_UPDATE, RECREATE, GREEN.
                            }
                        '''
                    )
                    echo "Response: ${response.content}"
                    if (response.status == 201) {
                        def jsonResponde = readJson text: response.content
                        DEPLOY_CODE = jsonResponde.code
                        echo "Deploy successful with code: ${env.DEPLOY_CODE}"
                        echo "Deploy Response: ${response.content}"
                    } else {
                        error "Build failed with status: ${response.status}"
                    }
                }
            }
        }
    }
}