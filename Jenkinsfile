
@'
pipeline {
    // Tous les stages seront exécutés sur l'agent Windows connecté à Docker.
    agent {
        label 'windows && docker'
    }

    // Évite le checkout automatique : nous le faisons explicitement dans le stage Checkout.
    options {
        skipDefaultCheckout(true)
    }

    environment {
        DOCKERHUB_USER = 'kalam77'
        FRONTEND_IMAGE = 'kalam77/bijouterie-frontend'
        BACKEND_IMAGE  = 'kalam77/bijouterie-backend'
        MYSQL_IMAGE    = 'kalam77/bijouterie-mysql'
    }

    stages {

        // STAGE 1 : récupération du code source depuis GitHub.
        stage('Checkout') {
            steps {
                checkout scm
                bat 'git status'
            }
        }

        // STAGE 2 : construction des trois images depuis leurs Dockerfiles.
        stage('Build') {
            steps {
                bat '''
                    docker build -f frontend/Dockerfile -t %FRONTEND_IMAGE%:build-%BUILD_NUMBER% frontend
                    docker build -f backend/Dockerfile -t %BACKEND_IMAGE%:build-%BUILD_NUMBER% backend
                    docker build -f database/Dockerfile -t %MYSQL_IMAGE%:build-%BUILD_NUMBER% database
                '''
            }
        }

        // STAGE 3 : attribution du numéro de build Jenkins comme tag final.
        stage('Tag') {
            steps {
                bat '''
                    docker tag %FRONTEND_IMAGE%:build-%BUILD_NUMBER% %FRONTEND_IMAGE%:%BUILD_NUMBER%
                    docker tag %BACKEND_IMAGE%:build-%BUILD_NUMBER% %BACKEND_IMAGE%:%BUILD_NUMBER%
                    docker tag %MYSQL_IMAGE%:build-%BUILD_NUMBER% %MYSQL_IMAGE%:%BUILD_NUMBER%
                '''
            }
        }

        // STAGE 4 : authentification sécurisée et publication sur Docker Hub.
        stage('Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USR',
                        passwordVariable: 'DOCKERHUB_PSW'
                    )
                ]) {
                    bat '''
                        powershell -NoProfile -NonInteractive -Command "$env:DOCKERHUB_PSW | docker login --username $env:DOCKERHUB_USR --password-stdin"
                        if errorlevel 1 exit /b 1

                        docker push %FRONTEND_IMAGE%:%BUILD_NUMBER%
                        if errorlevel 1 exit /b 1

                        docker push %BACKEND_IMAGE%:%BUILD_NUMBER%
                        if errorlevel 1 exit /b 1

                        docker push %MYSQL_IMAGE%:%BUILD_NUMBER%
                        if errorlevel 1 exit /b 1

                        docker logout
                    '''
                }
            }
        }
    }

    // Affiche un message final selon le résultat du pipeline.
    post {
        success {
            echo 'SUCCES : les trois images Bijouterie ont ete publiees sur Docker Hub.'
        }
        failure {
            echo 'ECHEC : verifier les logs du stage en erreur.'
        }
    }
}
'@ | Set-Content -Path Jenkinsfile -Encoding UTF8
