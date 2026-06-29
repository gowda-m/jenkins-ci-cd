pipeline {
 agent any

 environment {
 NGINX_VERSION = "1.31.1"
 OLD_VERSION = "1.20.1"
 NGINX_PATH = "/opt/nginx"
 BACKUP_PATH = "/opt/backup"
 }

 stages {
 stage('Backup Configs & SSL') {
 steps {
 sh '''
 mkdir -p ${BACKUP_PATH}
 cp ${NGINX_PATH}/conf/nginx.conf ${BACKUP_PATH}/nginx.conf
 cp -r ${NGINX_PATH}/conf/conf.d ${BACKUP_PATH}/conf.d
 cp -r ${NGINX_PATH}/ssl ${BACKUP_PATH}/ssl
 '''
 }
 }

 stage('Download & Build New Nginx') {
 steps {
 sh '''
 wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
 tar -xvzf nginx-${NGINX_VERSION}.tar.gz
 cd nginx-${NGINX_VERSION}
 ./configure --prefix=${NGINX_PATH} --with-http_ssl_module
 make
 sudo make install
 '''
 }
 }

 stage('Restore Configs & SSL') {
 steps {
 sh '''
 sudo cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/nginx.conf
 sudo cp -r ${BACKUP_PATH}/conf.d/* ${NGINX_PATH}/conf/conf.d/
 sudo cp -r ${BACKUP_PATH}/ssl/* ${NGINX_PATH}/ssl/
 '''
 }
 }

 stage('Validation via systemd') {
 steps {
 sh '''
 sudo systemctl daemon-reload
 sudo systemctl restart nginx
 sudo systemctl status nginx --no-pager
 curl -vk https://localhost | grep "200 OK"
 '''
 }
 }

 stage('Rollback') {
 when {
 expression { currentBuild.result == 'FAILURE' }
 }
 steps {
 sh '''
 echo "Rolling back to Nginx ${OLD_VERSION}"
 wget http://nginx.org/download/nginx-${OLD_VERSION}.tar.gz
 tar -xvzf nginx-${OLD_VERSION}.tar.gz
 cd nginx-${OLD_VERSION}
 ./configure --prefix=${NGINX_PATH} --with-http_ssl_module
 make
 sudo make install
 sudo cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/nginx.conf
 sudo cp -r ${BACKUP_PATH}/conf.d/* ${NGINX_PATH}/conf/conf.d/
 sudo cp -r ${BACKUP_PATH}/ssl/* ${NGINX_PATH}/ssl/
 sudo systemctl daemon-reload
 sudo systemctl restart nginx
 '''
 }
 }
 }

 post {
 success {
 emailext(
 subject: "Nginx Upgrade Successful",
 body: "Nginx ${NGINX_VERSION} upgrade completed successfully. Configs and SSL restored from ${BACKUP_PATH}. Service restarted via systemd.",
 to: "ops-team@example.com"
 )
 }
 failure {
 emailext(
 subject: "Nginx Upgrade Failed - Rolled Back",
 body: "Upgrade to Nginx ${NGINX_VERSION} failed. Rolled back to ${OLD_VERSION}. Configs and SSL restored from ${BACKUP_PATH}. Service restarted via systemd.",
 to: "ops-team@example.com"
 )
 }
 }
 }