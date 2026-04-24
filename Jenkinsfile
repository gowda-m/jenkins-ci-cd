pipeline {
    agent { label 'apache' }

    stages {

        stage('Deploy') {
            steps {
                sh '''
                echo "Deploying application..."

                sudo rm -rf /srv/www/htdocs/*
                sudo cp -r * /srv/www/htdocs/

                echo "Deployment completed successfully"
                '''
            }
        }
    }

    post {
        success {
            emailext (
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build SUCCESSFUL\nCheck: ${env.BUILD_URL}",
                to: "gowdaviji58@gmail.com"
            )
        }
        failure {
            emailext (
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build FAILED\nCheck: ${env.BUILD_URL}",
                to: "gowdaviji58@gmail.com"
            )
        }
    }
}