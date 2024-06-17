pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('TOKEN_CP1D')  // El ID de la credencial que creaste
        AWS_REGION = 'us-east-1' // Define tu región de AWS
        STACK_NAME = 'todo-list-aws-staging' // Nombre del stack de CloudFormation para staging
        S3_BUCKET = 'aws-sam-cli-managed-staging-samclisourcebucket-leyre' // Bucket S3 para el empaquetado de SAM en staging
        CAPABILITIES = 'CAPABILITY_IAM' // Capacidades necesarias para el despliegue
        STAGE = 'staging' // Define el valor del stage
    }
    
    options {
        skipDefaultCheckout(true) // Evita que Jenkins realice un checkout por defecto
        
    }
    
    stages {
        stage('Get Code') {
            steps {
                //Obtener codigo del repositorio
               checkout([$class: 'GitSCM', branches: [[name: '*/develop']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://${env.GITHUB_TOKEN}@github.com/leyrecanales10/todo-list-aws.git']]])
            }
        }
        
        stage('Static Test'){
            parallel{
                stage('Flake8'){
		            steps{
                        sh 'flake8 --exit-zero --format=pylint src > flake8.out'
			            recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 100, type: 'TOTAL', unstable: true], [threshold: 200, type: 'TOTAL', unstable: false]]
		                
		            }
		        }

	            stage('Bandit'){
	                steps{
                        sh 'bandit --exit-zero -r src -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"'
				        recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 100, type: 'TOTAL', unstable: true], [threshold: 200, type: 'TOTAL', unstable: false]]
			            
				    }
		        }
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
                BASE_URL=${BASE_URL} pytest test/integration/todoApiTest.py
                '''
                
            }
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
				git merge -X ours origin/develop || true
                git push origin master
                '''
            }
        }
        
    }
}