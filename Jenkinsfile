pipeline {
    agent any

    stages {
        // --- FASE 1: CI (Continuous Integration) ---
        stage('CI - Build & SonarQube Analysis') {
            steps {
                echo 'Iniciando análise de qualidade com SonarQube...'
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Verificando o "Quality Gate" no SonarQube...'
                timeout(time: 6, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // --- FASE 2: CD (Continuous Delivery/Deployment) ---
        stage('CD - Deploy to Development') {
            steps {
                echo 'Deploy para o ambiente de Desenvolvimento...'

                // 1. Para qualquer processo antigo que esteja usando a porta 8001 (NOVA PORTA)
                echo 'Parando servidor de desenvolvimento anterior (porta 8001)...'
                sh 'fuser -k 8001/tcp || true' // <-- VAI FUNCIONAR AGORA
                                               //     porque instalamos 'fuser'

                // 2. Copia os arquivos do site
                sh 'rm -rf /tmp/dev-webapp && mkdir -p /tmp/dev-webapp'
                sh 'cp -R * /tmp/dev-webapp'

                // 3. Inicia um servidor web Python 3 na porta 8001 (NOVA PORTA)
                echo 'Iniciando servidor web em http://<IP_DO_AGENTE_JENKINS>:8001'
                sh '''
                    cd /tmp/dev-webapp
                    nohup python3 -m http.server 8001 > /tmp/dev-server.log 2>&1 &
                ''' // <-- VAI FUNCIONAR AGORA
                    //     porque instalamos 'python3'
                echo 'App "publicado" em Desenvolvimento.'
            }
        }

        stage('Approval for Production') {
            steps {
                echo 'Aguardando aprovação manual para Produção.'
                input message: 'O ambiente de Desenvolvimento foi validado. Aprovar deploy para Produção?', ok: 'Sim, enviar para Produção'
            }
        }

        // --- FASE 3: CD (Continuous Deployment) ---
        stage('CD - Deploy to Production') {
            steps {
                echo 'Deploy para o ambiente de Produção...'

                // 1. Para qualquer processo antigo que esteja usando a porta 8002 (NOVA PORTA)
                echo 'Parando servidor de produção anterior (porta 8002)...'
                sh 'fuser -k 8002/tcp || true' // <-- VAI FUNCIONAR AGORA

                // 2. Copia os arquivos do site
                sh 'rm -rf /tmp/prod-webapp && mkdir -p /tmp/prod-webapp'
                sh 'cp -R * /tmp/prod-webapp'

                // 3. Inicia um servidor web Python 3 na porta 8002 (NOVA PORTA)
                echo 'Iniciando servidor web em http://<IP_DO_AGENTE_JENKINS>:8002'
                sh '''
                    cd /tmp/prod-webapp
                    nohup python3 -m http.server 8002 > /tmp/prod-server.log 2>&1 &
                ''' // <-- VAI FUNCIONAR AGORA
                echo 'App "publicado" em Produção!'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finalizada.'
        }
    }
}
