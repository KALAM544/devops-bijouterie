pipeline {
    // Utilisation de notre agent Windows avec Docker
    agent {
        label 'windows && docker'
    }

    // Desactivation du checkout automatique
    options {
        skipDefaultCheckout(true)
    }

    // Definition des noms des trois images Docker
    environment {
        FRONTEND_IMAGE = 'kalam77/bijouterie-frontend'
        BACKEND_IMAGE  = 'kalam77/bijouterie-backend'
        MYSQL_IMAGE    = 'kalam77/bijouterie-mysql'
    }

    stages {

        // ETAPE 1 : Recuperation du code depuis GitHub
        stage('Checkout') {
            steps {
                checkout scm
                bat 'git status'
            }
        }

        // ETAPE 2 : Construction des trois images Docker
        stage('Build') {
            steps {
                bat '''
                    docker build -f frontend/Dockerfile -t %FRONTEND_IMAGE%:build-%BUILD_NUMBER% frontend
                    docker build -f backend/Dockerfile -t %BACKEND_IMAGE%:build-%BUILD_NUMBER% backend
                    docker build -f database/Dockerfile -t %MYSQL_IMAGE%:build-%BUILD_NUMBER% database
                '''
            }
        }

        // ETAPE 3 : Attribution du numero de build Jenkins comme tag
        stage('Tag') {
            steps {
                bat '''
                    docker tag %FRONTEND_IMAGE%:build-%BUILD_NUMBER% %FRONTEND_IMAGE%:%BUILD_NUMBER%
                    docker tag %BACKEND_IMAGE%:build-%BUILD_NUMBER% %BACKEND_IMAGE%:%BUILD_NUMBER%
                    docker tag %MYSQL_IMAGE%:build-%BUILD_NUMBER% %MYSQL_IMAGE%:%BUILD_NUMBER%
                '''
            }
        }

        // ETAPE 4 : Connexion et publication des images sur Docker Hub
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

    // Affichage du resultat final du pipeline
    post {
        success {
            echo 'Succes : les trois images ont ete publiees sur Docker Hub.'
        }
        failure {
            echo 'Echec : verifier les erreurs dans la console Jenkins.'
        }
    }
}