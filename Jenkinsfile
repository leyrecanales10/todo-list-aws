pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('TOKEN_CP1D')  // El ID de la credencial que creaste
        AWS_REGION = 'us-east-1' // Define tu región de AWS
        STACK_NAME = 'todo-list-aws-production' // Nombre del stack de CloudFormation para production
        S3_BUCKET = 'aws-sam-cli-managed-production-samclisourcebucket-leyre' // Bucket S3 para el empaquetado de SAM en production
        CAPABILITIES = 'CAPABILITY_IAM' // Capacidades necesarias para el despliegue
        STAGE = 'production' // Define el valor del stage
    }
    
    options {
        skipDefaultCheckout(true) // Evita que Jenkins realice un checkout por defecto
        
    }
    
    stages {
        stage('Get Code') {
            steps {
                //Obtener codigo del repositorio
               checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://${env.GITHUB_TOKEN}@github.com/leyrecanales10/todo-list-aws.git']]])
            }
        }
	    
	    stage('Deploy'){
	        steps{
	            
	            echo "Construcción"
	            sh 'sam build'
	            
	            echo "Validación"
	            sh 'sam validate --template template.yaml --region ${AWS_REGION}'
	            
	            echo "Despliegue"
                sh "sam deploy --stack-name ${env.STACK_NAME} --s3-bucket ${env.S3_BUCKET} --capabilities ${env.CAPABILITIES} --region ${env.AWS_REGION} --no-confirm-changeset  --no-fail-on-empty-changeset --parameter-overrides Stage=${STAGE} --template-file template.yaml"
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
<<<<<<< HEAD
        }    
=======
        }
        
        stage('Promote') {
            steps {
				sh 'echo "Este es un cambio para asegurar que haya cambios en la rama" > cambio.txt'
                sh 'git add cambio.txt'
                
                sh 'git commit -m "Subimos y mergeamos"'
                
                
                
                // Agregar la URL del repositorio con el token de autenticación
                sh '''
                git remote set-url origin https://${GITHUB_TOKEN}@github.com/leyrecanales10/todo-list-aws.git
                git push origin HEAD:develop
                '''
                
                
                 // Checkout a la rama master y mergear
                sh '''
                git remote set-url origin https://${GITHUB_TOKEN}@github.com/leyrecanales10/todo-list-aws.git
                git clean -f
				git checkout master
                git pull origin master
				git merge --no-commit origin/develop || true
                git reset --Jenkinsfile || true
                git add .
				git commit -m "Merge y Elimina JenkinsfileCI durante merge"
                git push origin master
                '''
            }
        }
        
>>>>>>> origin/develop
    }
}