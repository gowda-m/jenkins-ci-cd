pipeline {
    agent { label 'apache' }
 
    stages {
 
        stage('Deploy') {
            steps {
                sh '''
                echo "Deploying application..."
 
                rm -rf /srv/www/htdocs/*
                cp -r * /srv/www/htdocs/
 
                echo "Deployment completed successfully"
                '''
            }
        }
 
    }
}