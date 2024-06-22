pipeline {
    agent any
    
    environment {
        GITHUB_TOKEN = credentials('TOKEN_CP1D')  // El ID de la credencial que creaste
        AWS_REGION = 'us-east-1' // Define tu región de AWS
        SAMCONFIG_PATH = 'config-repo/samconfig.toml' // Ruta al archivo samconfig.toml dentro del subdirectorio
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
                script {
                    // Clonar el repositorio principal en el subdirectorio 'main-repo'
                    sh '''
                    git clone -b develop https://${GITHUB_TOKEN}@github.com/leyrecanales10/todo-list-aws.git main-repo
                    '''
                   
                    // Clonar el repositorio de configuración en el subdirectorio 'config-repo'
                    sh '''
                    git clone -b staging https://${GITHUB_TOKEN}@github.com/leyrecanales10/todo-list-aws-config.git config-repo
                    '''
                }
            }
        }
        
        stage('Static Test') {
            parallel {
                stage('Flake8') {
                    steps {
                        dir('main-repo') {
                            sh 'flake8 --exit-zero --format=pylint src > flake8.out'
                            recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 100, type: 'TOTAL', unstable: true], [threshold: 200, type: 'TOTAL', unstable: false]]
                        }
                    }
                }

                stage('Bandit') {
                    steps {
                        dir('main-repo') {
                            sh 'bandit --exit-zero -r src -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"'
                            recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 100, type: 'TOTAL', unstable: true], [threshold: 200, type: 'TOTAL', unstable: false]]
                        }
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                dir('main-repo') {
                    echo "Construcción"
                    sh 'sam build'
                    
                    echo "Validación"
                    sh 'sam validate --template template.yaml --region ${AWS_REGION}'
                    
                    echo "Despliegue"
                    sh "sam deploy --config-file ${SAMCONFIG_PATH} --config-env staging --template-file template.yaml"
                }
                
                
            }
        }
        
        stage('Rest Test') {
            steps {
                dir('main-repo') {
                    // Obtener los outputs del stack usando AWS CLI y exportar la URL como una variable de entorno
                    sh 'export PYTHONPATH="$WORKSPACE"'
                    echo "Obtenemos la URL"
                    sh '''
                    #!/bin/bash
                    outputs=$(aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query "Stacks[0].Outputs")

                    # Extraer la URL específica del output (BaseUrlApi)
                    url=$(echo "$outputs" | grep '"OutputKey": "BaseUrlApi"' -A1 | grep '"OutputValue"' | cut -d '"' -f 4)

                    export BASE_URL=${url}

                    # Imprimir la URL para verificar
                    echo "Variable generada BASE_URL=${BASE_URL}"
                    
                    echo "Ejecutamos los test"
                    BASE_URL=${BASE_URL} pytest main-repo/test/integration/todoApiTest.py
                    '''
                }
            }
        }
        
        stage('Promote') {
            steps {
                dir('main-repo') {
                    sh 'echo "Este es un cambio para asegurar que haya cambios en la rama" > cambio3.txt'
                    sh 'git add cambio3.txt'
                    
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
                    cp Jenkinsfile Jenkinsfile_master
                    git merge --no-commit origin/develop || true
                    mv Jenkinsfile_master Jenkinsfile
                    git add .
                    git commit -m "Merge y Elimina JenkinsfileCI durante merge"
                    git push origin master
                    '''
                }
            }
        }
    }
}