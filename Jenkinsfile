pipeline {
    agent any

    parameters{
        choice(name: 'ENVIRONMENT', choices:[
            'dev',
            'stg',
            'prod'],
            description: 'Choose which environment to deploy to.')
        
        string(name: 'VERSION', description: 'Explicit version to deploy (i.e., "v0.1-51-g87b72a"). Leave blank to build latest commit')


        string(name: 'SUBSCRIPTION', defaultValue:'67d37eda-76c7-4c34-8503-2df917769cfd', description: ''' select subscription as:
            67d37eda-76c7-4c34-8503-2df917769cfd
            6fd0ed8f-99ff-480e-a5ff-f741fb2b8cec
            054168d1-524c-4d69-a9a2-bdf62c2bb47c''')

        string(name: 'RESOURCE_GROUP_NAME', defaultValue:'ssna-rg-cca-dev-eastus', description: ''' Azure Resource Group in which the FunctionApp need to deploy.
            ssna-rg-cca-dev-eastus
            ssna-rg-cca-stg-eastus
            ssna-rg-cca-prod-eastus
            ''')

        string(name: 'CONFIGURE_FUNCTIONAPP_NAME', defaultValue: 'ssna-func-cca-dev-eus-wfconfigure', description: '''The name of FunctionApp 
            ssna-func-cca-dev-eus-wfconfigure
            ssna-func-cca-stg-eus-wfconfigure
            ssna-func-cca-prod-eus-wfconfigure
            ''' )

        string(name: 'TRANSCODE_FUNCTIONAPP_NAME', defaultValue: 'ssna-func-cca-dev-eus-transcode', description: '''The name of FunctionApp 
            ssna-func-cca-dev-eus-transcode
            ssna-func-cca-stg-eus-transcode
            ssna-func-cca-prod-eus-transcode
            ''' )

        string(name: 'TRANSCRIBE_FUNCTIONAPP_NAME', defaultValue: 'ssna-func-cca-dev-eus-transcribe', description: '''The name of FunctionApp 
            ssna-func-cca-dev-eus-transcribe
            ssna-func-cca-stg-eus-transcribe
            ssna-func-cca-prod-eus-transcribe
            ''' )

        string(name: 'ANALYSE_FUNCTIONAPP_NAME', defaultValue: 'ssna-func-cca-dev-eus-wfanalyse', description: '''The name of FunctionApp 
            ssna-func-cca-dev-eus-wfanalyse
            ssna-func-cca-stg-eus-wfanalyse
            ssna-func-cca-prod-eus-wfanalyse
            ''' )

        string(name: 'REDACT_FUNCTIONAPP_NAME', defaultValue: 'ssna-func-cca-dev-eus-wfredact', description: '''The name of FunctionApp 
            ssna-func-cca-dev-eus-wfredact
            ssna-func-cca-stg-eus-wfredact
            ssna-func-cca-prod-eus-wfredact
            ''' )
        
        string(name: 'AZURE_LOGICAPP_NAME', defaultValue:'ssna-logiccapp-cca-dev-eastus', description: '''The name of LogicApp to deploy
            ssna-logiccapp-cca-dev-eastus
            ssna-logiccapp-cca-stg-eastus
            ssna-logiccapp-cca-prod-eastus
            ''' )
        
        string(name: 'SERVICE_BUS_TOPIC_NAME', defaultValue:'cca-dev-content', description: '''Name of the service bus topic
            cca-dev-content
            cca-stg-content
            cca-prod-content
            ''' )
    }

    environment {
        AZURE_CLIENT_ID = credentials("az_cca_${params.ENVIRONMENT}_client_id")
        AZURE_CLIENT_SECRET = credentials("az_cca_${params.ENVIRONMENT}_secret_value")
        AZURE_TENANT_ID = credentials("az_cca_${params.ENVIRONMENT}_tenant_id")
    }

    stages {
        stage('Checkout') {
            steps {
                // checkout scm
                git branch: 'feature/with_parameter', url: 'https://github.com/tajuddin-sonata/az-logicapp.git'

            }
        }

        
        stage ('replace variable in code') {
            steps {
                script {
                    sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID'
                    sh "az account set --subscription ${params.SUBSCRIPTION}"
                    
                    def configure_func_url = sh(script: "func azure functionapp list-functions ${params.CONFIGURE_FUNCTIONAPP_NAME} --show-keys | awk '/Invoke url:/ {print \$3}'", returnStdout: true).trim()
                    echo "configure_func_url: ${configure_func_url}"

                    def transcode_func_url = sh(script: "func azure functionapp list-functions ${params.TRANSCODE_FUNCTIONAPP_NAME} --show-keys | awk '/Invoke url:/ {print \$3}'", returnStdout: true).trim()
                    echo "transcode_func_url: ${transcode_func_url}"
                    
                    def transcribe_func_url = sh(script: "func azure functionapp list-functions ${params.TRANSCRIBE_FUNCTIONAPP_NAME} --show-keys | awk '/Invoke url:/ {print \$3}'", returnStdout: true).trim()
                    echo "transcribe_func_url: ${transcribe_func_url}"
                    
                    def analyse_func_url = sh(script: "func azure functionapp list-functions ${params.ANALYSE_FUNCTIONAPP_NAME} --show-keys | awk '/Invoke url:/ {print \$3}'", returnStdout: true).trim()
                    echo "analyse_func_url: ${analyse_func_url}"
                    
                    def redact_func_url = sh(script: "func azure functionapp list-functions ${params.REDACT_FUNCTIONAPP_NAME} --show-keys | awk '/Invoke url:/ {print \$3}'", returnStdout: true).trim()
                    echo "redact_func_url: ${redact_func_url}"
                    
                    sh """
                        cd src/
                        sed -i 's|\$CONFIGURE_FUNC_URL|${configure_func_url}|g' parameters.json
                        sed -i 's|\$TRANSCODE_FUNC_URL|${transcode_func_url}|g' parameters.json
                        sed -i 's|\$TRANSCRIBE_FUNC_URL|${transcribe_func_url}|g' parameters.json
                        sed -i 's|\$ANALYSE_FUNC_URL|${analyse_func_url}|g' parameters.json
                        sed -i 's|\$REDACT_FUNC_URL|${redact_func_url}|g' parameters.json
                        sed -i 's|\$SERVICE_BUS_TOPIC_NAME|${params.SERVICE_BUS_TOPIC_NAME}|g' parameters.json

                    """
                }
            }
        }

        stage('Deploy artifact to Nexus & LogicApp') {
            steps {
                script {
                    echo "Deploy artifact to Nexus"
                    def ver = params.VERSION
                    sh """
                        #!/bin/bash
                
                        if [ -z "$ver" ]; then
                            artifact_version=\$(git describe --tags)
                            echo "\${artifact_version}" > src/version.txt
                            cd src/
                            zip -r "../az-ci-trail-workflow-orchestrator-\${artifact_version}.zip" *
                            cd $WORKSPACE
                            echo "CREATED [az-ci-trail-workflow-orchestrator-\${artifact_version}.zip]"
                            curl -v -u deployment:deployment123 --upload-file \
                                "az-ci-trail-workflow-orchestrator-\${artifact_version}.zip" \
                                "http://172.190.171.178:8081/repository/packages/cca/az-ci-trail-workflow-orchestrator-\${artifact_version}.zip"
                        else
                            artifact_version=$ver
                            echo "Downloading specified artifact version from Nexus..."
                            curl -v -u nexus-user:nexus@123 -O "http://172.190.171.178:8081/repository/packages/cca/az-ci-trail-workflow-orchestrator-\${artifact_version}.zip"
                        fi
                        rm -rf "az-ci-trail-workflow-orchestrator-\${artifact_version}"
                        unzip "az-ci-trail-workflow-orchestrator-\${artifact_version}.zip" -d "az-ci-trail-workflow-orchestrator-\${artifact_version}"

                        ls -ltr
                        az webapp deployment source config-zip -g ${params.RESOURCE_GROUP_NAME} -n ${params.AZURE_LOGICAPP_NAME} --subscription ${params.SUBSCRIPTION} --src az-ci-trail-workflow-orchestrator-\${artifact_version}.zip
                    """
                }
            }
        }

    }
    post {
        success {
            cleanWs()
        }
    }
}

