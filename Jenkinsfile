pipeline {
    // Utilisation de notre agent Windows avec Docker
    agent {
        label 'windows && docker'
    }

    // Desactivation du checkout automatique
    options {
        skipDefaultCheckout(true)
    }

    // Noms des trois images Docker
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

        // ETAPE 3 : Attribution du numero de build Jenkins
        stage('Tag') {
            steps {
                bat '''
                    docker tag %FRONTEND_IMAGE%:build-%BUILD_NUMBER% %FRONTEND_IMAGE%:%BUILD_NUMBER%
                    docker tag %BACKEND_IMAGE%:build-%BUILD_NUMBER% %BACKEND_IMAGE%:%BUILD_NUMBER%
                    docker tag %MYSQL_IMAGE%:build-%BUILD_NUMBER% %MYSQL_IMAGE%:%BUILD_NUMBER%
                '''
            }
        }

        // ETAPE 4 : Publication des images sur Docker Hub
        stage('Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credential',
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

        // ETAPE 5 : Mise a jour des manifestes Kubernetes
        stage('Update Manifests') {
            steps {
                bat '''
                    powershell -NoProfile -Command "$p='k8s/02-mysql-deployment.yaml'; $c=Get-Content $p -Raw; $c=[regex]::Replace($c,'(?m)^(\\s*image:\\s*).+$','$1kalam77/bijouterie-mysql:%BUILD_NUMBER%'); [IO.File]::WriteAllText((Resolve-Path $p),$c,(New-Object Text.UTF8Encoding($false)))"

                    powershell -NoProfile -Command "$p='k8s/06-backend-deployment.yaml'; $c=Get-Content $p -Raw; $c=[regex]::Replace($c,'(?m)^(\\s*image:\\s*).+$','$1kalam77/bijouterie-backend:%BUILD_NUMBER%'); [IO.File]::WriteAllText((Resolve-Path $p),$c,(New-Object Text.UTF8Encoding($false)))"

                    powershell -NoProfile -Command "$p='k8s/09-frontend-deployment.yaml'; $c=Get-Content $p -Raw; $c=[regex]::Replace($c,'(?m)^(\\s*image:\\s*).+$','$1kalam77/bijouterie-frontend:%BUILD_NUMBER%'); [IO.File]::WriteAllText((Resolve-Path $p),$c,(New-Object Text.UTF8Encoding($false)))"

                    git diff -- k8s/
                    git config user.name "Jenkins"
                    git config user.email "jenkins@localhost"
                    git add k8s/02-mysql-deployment.yaml k8s/06-backend-deployment.yaml k8s/09-frontend-deployment.yaml
                    git commit -m "Mise a jour des images Kubernetes - build %BUILD_NUMBER%"
                '''

                // Envoi des manifestes modifies sur GitHub
                withCredentials([
                    gitUsernamePassword(
                        credentialsId: 'github-credential',
                        gitToolName: 'Default'
                    )
                ]) {
                    bat 'git push origin HEAD:main'
                }
            }
        }
    }

    // Affichage du resultat final
    post {
        success {
            echo 'Succes : images publiees et manifestes Kubernetes mis a jour sur GitHub.'
        }
        failure {
            echo 'Echec : verifier les erreurs dans la console Jenkins.'
        }
    }
}