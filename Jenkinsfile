pipeline {
    agent { label 'apache' }
 
    stages {
        stage('Deploy') {
            steps {
                sh '''
                echo "Deploy application..."
 
                sudo rm -rf /srv/www/htdocs/*
                sudo cp -r * /srv/www/htdocs/
 
                echo "Deployment completed successfully"
                '''
            }
        }
    }
 
    post {
        success {
            emailext(
                to: 'gowda.m@intelizign.com,iliyas.bhatari@intelizign.com,puneetkumar.neelanjanmath@intelizign.com',
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/html',
                body: """
<html>
<body>
<h2>✅ Jenkins Build Successful</h2>
 
<p><b>Job Name:</b> ${env.JOB_NAME}</p>
<p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
<p><b>Build Status:</b> SUCCESS</p>
 
<p>
<b>Build URL:</b><br>
<a href="${env.BUILD_URL}">
${env.BUILD_URL}
</a>
</p>
 
</body>
</html>
"""
            )
        }
 
        failure {
            emailext(
                to: 'gowda.m@intelizign.com,iliyas.bhatari@intelizign.com,puneetkumar.neelanjanmath@intelizign.com',
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/html',
                body: """
<html>
<body>
<h2>❌ Jenkins Build Failed</h2>
 
<p><b>Job Name:</b> ${env.JOB_NAME}</p>
<p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
<p><b>Build Status:</b> FAILED</p>
 
<p>
<b>Build URL:</b><br>
<a href="${env.BUILD_URL}">
${env.BUILD_URL}
</a>
</p>
 
</body>
</html>
"""
            )
        }
    }
}
 
