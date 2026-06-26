pipeline {
 agent any

 environment {
 NGINX_VERSION = "1.31.1"
 TARBALL = "/opt/nginx/nginx-${NGINX_VERSION}.tar.gz"
 BACKUP_PATH = "/opt/backup"
 }

 stages {

 stage('Backup Nginx Config') {
 steps {
 sh '''
 echo "Taking backup of Nginx config..."

 sudo mkdir -p ${BACKUP_PATH}
 sudo cp -a /etc/nginx ${BACKUP_PATH}/nginx_backup

 echo "Backup completed"
 '''
 }
 }

 stage('Build & Install Nginx') {
 steps {
 sh '''
 echo "Starting Nginx upgrade build..."

 cd /tmp
 rm -rf nginx-${NGINX_VERSION}

 tar -xzf ${TARBALL}
 cd nginx-${NGINX_VERSION}

 ./configure \
 --prefix=/etc/nginx \
 --sbin-path=/usr/sbin/nginx \
 --conf-path=/etc/nginx/nginx.conf \
 --with-http_ssl_module

 make
 sudo make install

 echo "Nginx install completed"
 '''
 }
 }

 stage('Validate & Restart Nginx') {
 steps {
 sh '''
 echo "Validating Nginx configuration..."

 sudo nginx -t

 echo "Restarting Nginx..."
 sudo systemctl restart nginx

 echo "Current Nginx version:"
 nginx -v
 '''
 }
 }
 }

 post {

 success {
 emailext(
 to: "gowda.m@intelizign.com",
 subject: "SUCCESS: Nginx ${NGINX_VERSION} Upgrade",
 body: """
 Nginx upgrade completed successfully.

Version: ${NGINX_VERSION}
 Server: ${env.NODE_NAME}
 """
 )
 }

 failure {
 sh '''
 echo "Rollback started..."

 sudo rm -rf /etc/nginx
 sudo cp -a ${BACKUP_PATH}/nginx_backup /etc/nginx

 sudo systemctl restart nginx

 echo "Rollback completed"
 '''

 emailext(
 to: "gowda.m@intelizign.com",
 subject: "FAILED: Nginx Upgrade Rolled Back",
 body: """
 Nginx upgrade FAILED and rollback executed.

Reverted to previous configuration.
 """
 )
 }
 }
 }