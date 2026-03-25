pipeline {

    agent { label 'apache' }
 
    stages {
 
        stage('Deploy') {

            steps {

                sh '''

                echo "Deploying application..."
 
                # Clean old files

                rm -rf /var/www/html/*
 
                # Copy new files

                cp -r * /var/www/html/
 
                echo "Deployment completed successfully"

                '''

            }

        }
 
    }

}
 