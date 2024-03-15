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
        
        string(name: 'VERSION', description: 'Explicit version to deploy (i.e., "v0.1"). Leave blank to build latest commit')


        string(name: 'SUBSCRIPTION', defaultValue:'48986b2e-5349-4fab-a6e8-d5f02072a4b8', description: ''' select subscription as:
            48986b2e-5349-4fab-a6e8-d5f02072a4b8
            34b1c36e-d8e8-4bd5-a6f3-2f92a1c0626e
            70c3af66-8434-419b-b808-0b3c0c4b1a04''')

        string(name: 'RESOURCE_GROUP_NAME', defaultValue:'sitl-rg-prod-eus-cca', description: ''' Azure Resource Group in which the FunctionApp need to deploy.
            sitl-rg-dev-eus-cca
            sitl-rg-stg-eus-cca
            sitl-rg-prod-eus-cca
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
        
        string(name: 'AZURE_LOGICAPP_NAME', defaultValue:'ssna-logicapp-cca-dev-eastus', description: '''The name of LogicApp to deploy
            ssna-logicapp-cca-dev-eastus
            ssna-logicapp-cca-stg-eastus
            ssna-logicapp-cca-prod-eastus
            ''' )

        string(name: 'AZURE_STT_NAME', defaultValue:'ssna-cogs-cca-dev-eastus', description: '''The name of LogicApp to deploy
            ssna-cogs-cca-dev-eastus
            ssna-cogs-cca-stg-eastus
            ssna-cogs-cca-prod-eastus
            ''' )
        string(name: 'STT_RG_NAME', defaultValue:'sitl-rg-prod-eus-cca', description: ''' Azure Resource Group name of Azure STT service.
            sitl-rg-dev-eus-cca
            sitl-rg-stg-eus-cca
            sitl-rg-prod-eus-cca
            ''')

        string(name: 'SERVICE_BUS_TOPIC_NAME', defaultValue:'ssna-cca-dev-contentt', description: '''Name of the service bus topic
            ssna-cca-dev-content
            ssna-cca-stg-content
            ssna-cca-prod-content
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

                    def azure_stt_endpoint = sh(script: "az cognitiveservices account show -n ${params.AZURE_STT_NAME} -g ${params.STT_RG_NAME} --query properties.endpoint --output tsv | awk -F/ '{print \$3}'", returnStdout: true).trim()
                    echo "azure_stt_endpoint: ${azure_stt_endpoint}"

                    def azure_stt_key = sh(script: "az cognitiveservices account keys list --name ${params.AZURE_STT_NAME} --resource-group ${params.STT_RG_NAME} --query key1 --output tsv", returnStdout: true).trim()
                    echo "azure_stt_key: ${azure_stt_key}"

                    sh """
                        cd src/STT-Workflow/
                        sed -i 's|\$CONFIGURE_FUNC_URL|${configure_func_url}|g' workflow.json
                        sed -i 's|\$TRANSCODE_FUNC_URL|${transcode_func_url}|g' workflow.json
                        sed -i 's|\$TRANSCRIBE_FUNC_URL|${transcribe_func_url}|g' workflow.json
                        sed -i 's|\$ANALYSE_FUNC_URL|${analyse_func_url}|g' workflow.json
                        sed -i 's|\$REDACT_FUNC_URL|${redact_func_url}|g' workflow.json
                        sed -i 's|\$AZURE_STT_ENDPOINT|${azure_stt_endpoint}|g' workflow.json
                        sed -i 's|\$AZURE_STT_KEY|${azure_stt_key}|g' workflow.json
                        sed -i 's|\$SERVICE_BUS_TOPIC_NAME|${params.SERVICE_BUS_TOPIC_NAME}|g' workflow.json
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
                        az webapp deployment source config-zip -g ${params.RESOURCE_GROUP_NAME} -n ${params.AZURE_LOGICAPP_NAME} --subscription ${params.SUBSCRIPTION} --src az-ci-stt-workflow-orchestrator-\${artifact_version}.zip
                    """
                }
            }
        }

    }
}
