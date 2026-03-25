pipeline {
    agent { label 'apache' }

    stages {

        stage('Clone') {
            steps {
                git 'https://github.com/gowda-m/jenkins-ci-cd.git'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "Deploying..."

                sudo rm -rf /var/www/html/*
                sudo cp -r * /var/www/html/

                echo "Deployment Done"
                '''
            }
        }

    }
}