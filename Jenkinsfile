// ESTE É O CONTEÚDO CORRETO PARA O SEU ARQUIVO Jenkinsfile
pipeline {
    agent any 

    // Define o scanner que vamos usar
    tools {
        // 'SonarScanner' é o nome que você deu em "Global Tool Configuration"
        tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
    }
    
    stages {
        // O checkout é feito automaticamente pelo Jenkins
        // quando usamos "Pipeline script from SCM",
        // por isso NÃO TEMOS um estágio de 'Checkout'.

        stage('CI - Build & SonarQube Analysis') {
            steps {
                echo 'Iniciando análise de qualidade com SonarQube...'
                // 'SonarQube' é o nome que demos ao servidor na "Configure System"
                withSonarQubeEnv('SonarQube') {
                    // O repositório MobEAD já tem um arquivo 'sonar-project.properties'
                    sh "sonar-scanner"
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                echo 'Verificando o "Quality Gate" no SonarQube...'
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // --- FASE 2: CD (Continuous Delivery/Deployment) ---
        stage('CD - Deploy to Development') {
            steps {
                echo 'Deploy para o ambiente de Desenvolvimento...'
                sh 'rm -rf /tmp/dev-webapp && mkdir -p /tmp/dev-webapp'
                sh 'cp -R * /tmp/dev-webapp'
                echo 'App "publicado" em Desenvolvimento.'
            }
        }

        stage('Approval for Production') {
            steps {
                echo 'Aguardando aprovação manual para Produção.'
                input message: 'O ambiente de Desenvolvimento foi validado. Aprovar deploy para Produção?', ok: 'Sim, enviar para Produção'
            }
        }

        stage('CD - Deploy to Production') {
            steps {
                echo 'Deploy para o ambiente de Produção...'
                sh 'rm -rf /tmp/prod-webapp && mkdir -p /tmp/prod-webapp'
                sh 'cp -R * /tmp/prod-webapp'
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
