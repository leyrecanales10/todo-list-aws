pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('TOKEN_CP1D')  // El ID de la credencial que creaste
        AWS_REGION = 'us-east-1' // Define tu región de AWS
        SAMCONFIG_PATH = 'samconfig.toml'
    }
    
    options {
        skipDefaultCheckout(true) // Evita que Jenkins realice un checkout por defecto
        
    }
    
    stages {
        stage('Get Code') {
            steps {
                //Obtener codigo del repositorio
               checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://${env.GITHUB_TOKEN}@github.com/leyrecanales10/todo-list-aws.git']]])
               
               //Obtener codigo del repositorio configuracion
               checkout([$class: 'GitSCM', branches: [[name: '*/production']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://${env.GITHUB_TOKEN}@github.com/leyrecanales10/todo-list-aws-config.git']]])
            }
        }
	    
	    stage('Deploy'){
	        steps{
	            
	            echo "Construcción"
	            sh 'sam build'
	            
	            echo "Validación"
	            sh 'sam validate --template template.yaml --region ${AWS_REGION}'
	            
	            echo "Despliegue"
                sh "sam deploy --config-file ${SAMCONFIG_PATH} --config-env production --template-file template.yaml"
		    }
	        
	    }
	    
	    stage('Rest Test') {
            steps {
                // Obtener los outputs del stack usando AWS CLI y exportar la URL como una variable de entorno
                sh 'export PYTHONPATH="$WORKSPACE"'
                echo "Obtenemos la url"
                sh '''
                #!/bin/bash
                outputs=$(aws cloudformation describe-stacks --stack-name ${STACK_NAME} --query "Stacks[0].Outputs")

                # Extraer la URL específica del output (BaseUrlApi)
                url=$(echo "$outputs" | grep '"OutputKey": "BaseUrlApi"' -A1 | grep '"OutputValue"' | cut -d '"' -f 4)

                export BASE_URL=${url}
                

                # Imprimir la URL para verificar
                echo "Variable generada BASE_URL=${BASE_URL}"
                
                echo "Ejecutamos los test"
                BASE_URL=${BASE_URL} pytest test/integration/todoApiTest.py -k "test_api_gettodo or test_api_listtodos"
                '''
                
            }
        }    
    }
}