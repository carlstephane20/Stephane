# Stephane

pipeline {
    agent any

    environment {
        // Exemple : mettre ici les variables d'environnement
        DEPLOY_USER = 'your_user'
        DEPLOY_HOST = 'your.server.com'
        DEPLOY_PATH = '/var/www/your-app'
    }

    stages {
        stage('Cloner le code') {
            steps {
                git branch: 'main', url: 'git@github.com:your-org/your-repo.git'
            }
        }

        stage('Installer les dépendances') {
            steps {
                sh 'npm install'
            }
        }

        stage('Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Déploiement') {
            steps {
                sshagent (credentials: ['jenkins-ssh-key']) {
                    sh """
                        ssh $DEPLOY_USER@$DEPLOY_HOST 'mkdir -p $DEPLOY_PATH'
                        scp -r ./dist/* $DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH/
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Déploiement réussi !'
        }
        failure {
            echo 'Le pipeline a échoué.'
        }
    }
}
