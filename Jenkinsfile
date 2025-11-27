pipeline {
    agent any
    
    environment {
        FLASK_APP = 'vulnerable_app (2).py'
        FLASK_ENV = 'development'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Python') {
            steps {
                script {
                    // Instalar Python si es necesario (depende de tu entorno Jenkins)
                    sh 'python --version || echo "Python not installed"'
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '''
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        
        stage('Code Analysis') {
            steps {
                script {
                    // Análisis básico de código (opcional)
                    sh 'python -m py_compile "${FLASK_APP}" || echo "Compilation check completed"'
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    // Escaneo básico de seguridad (puedes agregar herramientas como bandit)
                    echo "Running basic security checks..."
                    // sh 'pip install bandit'
                    // sh 'bandit -r . -f html -o security_report.html || true'
                }
            }
        }
        
        stage('Test Application') {
            steps {
                script {
                    // Verificar que la aplicación puede iniciarse
                    sh '''
                        # Verificar que el archivo principal existe
                        if [ ! -f "${FLASK_APP}" ]; then
                            echo "ERROR: Application file not found!"
                            exit 1
                        fi
                        
                        # Verificar sintaxis básica de Python
                        python -m py_compile "${FLASK_APP}"
                        echo "✓ Application syntax is valid"
                    '''
                }
            }
        }
        
        stage('Deploy to Test') {
            steps {
                script {
                    echo "Deploying application to test environment..."
                    // Aquí puedes agregar pasos específicos de despliegue
                    // como copiar archivos a un servidor de prueba
                    
                    // Ejemplo: Iniciar la aplicación en background para testing
                    sh '''
                        nohup python "${FLASK_APP}" > flask_app.log 2>&1 &
                        echo $! > flask_app.pid
                        sleep 5
                        
                        # Verificar que la aplicación está corriendo
                        if ps -p $(cat flask_app.pid) > /dev/null; then
                            echo "✓ Application started successfully"
                            # Detener la aplicación después de la verificación
                            kill $(cat flask_app.pid)
                        else
                            echo "✗ Application failed to start"
                            exit 1
                        fi
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Limpieza: asegurarse de que la aplicación se detenga
                sh '''
                    if [ -f flask_app.pid ]; then
                        kill $(cat flask_app.pid) 2>/dev/null || true
                        rm -f flask_app.pid
                    fi
                    rm -f flask_app.log 2>/dev/null || true
                '''
            }
            
            // Publicar reportes (opcional)
            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: '',
                reportFiles: 'security_report.html',
                reportName: 'Security Report'
            ])
        }
        
        success {
            echo 'Pipeline completed successfully!'
        }
        
        failure {
            echo 'Pipeline failed!'
        }
    }
}