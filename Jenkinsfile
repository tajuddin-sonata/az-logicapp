pipeline {
    agent {
        label 'GCP-jenkins-worker04'
    }

    parameters{
        choice(name: 'ENVIRONMENT', choices:[
            'dev',
            'staging',
            'prod'],
            description: 'Choose which environment to deploy to.')
        
        string(name: 'VERSION', description: 'Explicit version to deploy (i.e., "v0.1-51-g87b72a"). Leave blank to build latest commit')


        string(name: 'SUBSCRIPTION', defaultValue:'48986b2e-5349-4fab-a6e8-d5f02072a4b8', description: ''' select subscription as:
            48986b2e-5349-4fab-a6e8-d5f02072a4b8
            34b1c36e-d8e8-4bd5-a6f3-2f92a1c0626e
            70c3af66-8434-419b-b808-0b3c0c4b1a04''')

        string(name: 'RESOURCE_GROUP_NAME', defaultValue:'ssna-rg-cca-dev-eus', description: ''' Azure Resource Group in which the FunctionApp need to deploy.
            ssna-rg-cca-dev-eus
            ssna-rg-cca-stg-eus
            ssna-rg-cca-prod-eus
            ''')

        string(name: 'CONFIGURE_FUNCTIONAPP_NAME', defaultValue: 'ssna-func-cca-dev-eastus-wfconfigure', description: '''The name of FunctionApp 
            ssna-func-cca-dev-eastus-wfconfigure
            ssna-func-cca-stg-eastus-wfconfigure
            ssna-func-cca-prd-eastus-wfconfigure
            ''' )

        string(name: 'TRANSCODE_FUNCTIONAPP_NAME', defaultValue: 'ssna-func-cca-dev-eastus-wftranscode', description: '''The name of FunctionApp 
            ssna-func-cca-dev-eastus-wftranscode
            ssna-func-cca-stg-eastus-wftranscode
            ssna-func-cca-prod-eastus-wftranscode
            ''' )

        string(name: 'TRANSCRIBE_FUNCTIONAPP_NAME', defaultValue: 'ssna-func-cca-dev-eastus-wftranscribe', description: '''The name of FunctionApp 
            ssna-func-cca-dev-eastus-wftranscribe
            ssna-func-cca-stg-eastus-wftranscribe
            ssna-func-cca-prod-eastus-wftranscribe
            ''' )

        string(name: 'ANALYSE_FUNCTIONAPP_NAME', defaultValue: 'ssna-func-cca-dev-eastus-wfanalyse', description: '''The name of FunctionApp 
            ssna-func-cca-dev-eastus-wfanalyse
            ssna-func-cca-stg-eastus-wfanalyse
            ssna-func-cca-prod-eastus-wfanalyse
            ''' )

        string(name: 'REDACT_FUNCTIONAPP_NAME', defaultValue: 'ssna-func-cca-dev-eastus-wfredact', description: '''The name of FunctionApp 
            ssna-func-cca-dev-eastus-wfredact
            ssna-func-cca-stg-eastus-wfredact
            ssna-func-cca-prod-eastus-wfredact
            ''' )
        
        string(name: 'AZURE_LOGICAPP_NAME', defaultValue:'ssna-logicapp-cca-dev-eastus', description: '''The name of LogicApp to deploy
            ssna-logicapp-cca-dev-eastus
            ssna-logicapp-cca-stg-eastus
            ssna-logicapp-cca-prod-eastus
            ''' )

        string(name: 'AZURE_STT_NAME', defaultValue:'CCA-Dev-Azure-AI-MultiService', description: '''The name of LogicApp to deploy
            CCA-Dev-Azure-AI-MultiService
            ''' )
        string(name: 'STT_RG_NAME', defaultValue:'CCA-DEV', description: ''' Azure Resource Group name of Azure STT service.
            CCA-DEV
            ''')
    }

    environment {
        AZURE_CLIENT_ID = credentials('azurerm_client_id')
        AZURE_CLIENT_SECRET = credentials('azurerm_client_secret')
        AZURE_TENANT_ID = credentials('azurerm_tenant_id')
    }

    stages {
        stage('Checkout') {
            steps {
                // checkout scm
                git branch: 'stt', url: 'https://github.com/tajuddin-sonata/az-logicapp.git'

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

                    def azure_stt_key = sh(script: "az cognitiveservices account keys list --name ${params.AZURE_STT_NAME} --resource-group ${params.STT_RG_NAME}", returnStdout: true).trim()
                    echo "redact_func_url: ${redact_func_url}"

                    sh """
                        cd src/STT-Workflow/
                        sed -i 's|\$CONFIGURE_FUNC_URL|${configure_func_url}|g' workflow.json
                        sed -i 's|\$TRANSCODE_FUNC_URL|${transcode_func_url}|g' workflow.json
                        sed -i 's|\$TRANSCRIBE_FUNC_URL|${transcribe_func_url}|g' workflow.json
                        sed -i 's|\$ANALYSE_FUNC_URL|${analyse_func_url}|g' workflow.json
                        sed -i 's|\$REDACT_FUNC_URL|${redact_func_url}|g' workflow.json
                        sed -i 's|\$AZURE_STT_NAME|${params.AZURE_STT_NAME}|g' workflow.json
                        sed -i 's|\$AZURE_STT_KEY|${azure_stt_key}|g' workflow.json
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
                            zip -r "../az-ci-stt-workflow-orchestrator-\${artifact_version}.zip" *
                            cd $WORKSPACE
                            echo "CREATED [az-ci-stt-workflow-orchestrator-\${artifact_version}.zip]"
                            curl -v -u deployment:deployment123 --upload-file \
                                "az-ci-stt-workflow-orchestrator-\${artifact_version}.zip" \
                                "http://74.225.187.237:8081/repository/packages/cca/az-ci-stt-workflow-orchestrator-\${artifact_version}.zip"
                        else
                            artifact_version=$ver
                            echo "Downloading specified artifact version from Nexus..."
                            curl -v -u nexus-user:nexus@123 -O "http://74.225.187.237:8081/repository/packages/cca/az-ci-stt-workflow-orchestrator-\${artifact_version}.zip"
                        fi
                        rm -rf "az-ci-stt-workflow-orchestrator-\${artifact_version}"
                        unzip "az-ci-stt-workflow-orchestrator-\${artifact_version}.zip" -d "az-ci-stt-workflow-orchestrator-\${artifact_version}"

                        ls -ltr
                        az logicapp deployment source config-zip -g ${params.RESOURCE_GROUP_NAME} -n ${params.AZURE_LOGICAPP_NAME} --subscription ${params.SUBSCRIPTION} --src az-ci-stt-workflow-orchestrator-\${artifact_version}.zip
                    """
                }
            }
        }

    }
}
