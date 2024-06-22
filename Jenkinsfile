pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('TOKEN_CP1D')  // El ID de la credencial que creaste
        AWS_REGION = 'us-east-1' // Define tu región de AWS
        SAMCONFIG_PATH = 'config-repo/samconfig.toml'
    }
    
    options {
        skipDefaultCheckout(true) // Evita que Jenkins realice un checkout por defecto
        
    }
    
    stages {
        stage('Limpieza agente principal'){
            steps{
				deleteDir()

            }
        }
        
        stage('Get Code') {
            steps {
                // Clonar el repositorio principal en el subdirectorio 'main-repo'
                sh '''
                    git clone -b master https://${GITHUB_TOKEN}@github.com/leyrecanales10/todo-list-aws.git main-repo
                '''
                   
                // Clonar el repositorio de configuración en el subdirectorio 'config-repo'
                sh '''
                    git clone -b production https://${GITHUB_TOKEN}@github.com/leyrecanales10/todo-list-aws-config.git config-repo
                '''
               }
        }
	    
	    stage('Deploy'){
	        steps{
	            dir('main-repo') {
                    echo "Construcción"
                    sh 'sam build'
                    
                    echo "Validación"
                    sh 'sam validate --template template.yaml --region ${AWS_REGION}'
                    
                    echo "Despliegue"
                    sh "sam deploy --config-file ../${SAMCONFIG_PATH} --config-env production --template-file template.yaml"
                }
		    }
	        
	    }
	    
	    stage('Rest Test') {
            steps {
                dir('main-repo') {
                    // Obtener los outputs del stack usando AWS CLI y exportar la URL como una variable de entorno
                    sh 'export PYTHONPATH="$WORKSPACE"'
                    echo "Obtenemos la url"
                    sh '''
                    #!/bin/bash
                    outputs=$(aws cloudformation describe-stacks --stack-name todo-list-aws-production --query "Stacks[0].Outputs")

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
}