pipeline {
 agent any

 environment {
 NGINX_VERSION = "1.31.1"
 OLD_VERSION = "1.20.1"
 NGINX_PATH = "/opt/nginx"
 BACKUP_PATH = "/opt/backup"
 }

 stages {

 stage('Backup Configurations') {
 steps {
 sh '''
 set -e

 sudo mkdir -p ${BACKUP_PATH}

 echo "===== Verifying Nginx Directories ====="
 whoami
 hostname
 ls -ld ${NGINX_PATH}
 ls -ld ${NGINX_PATH}/conf

 # Backup nginx.conf
 if [ -f ${NGINX_PATH}/conf/nginx.conf ]; then
 echo "Backing up nginx.conf..."
 sudo cp ${NGINX_PATH}/conf/nginx.conf ${BACKUP_PATH}/
 else
 echo "ERROR: nginx.conf not found!"
 exit 1
 fi

 # Backup conf.d if exists
 if [ -d ${NGINX_PATH}/conf/conf.d ]; then
 echo "Backing up conf.d..."
 sudo cp -r ${NGINX_PATH}/conf/conf.d ${BACKUP_PATH}/
 else
 echo "INFO: conf.d directory not found. Skipping."
 fi

 # Backup ssl if exists
 if [ -d ${NGINX_PATH}/ssl ]; then
 echo "Backing up SSL certificates..."
 sudo cp -r ${NGINX_PATH}/ssl ${BACKUP_PATH}/
 else
 echo "INFO: SSL directory not found. Skipping."
 fi
 '''
 }
 }

 stage('Download & Build Nginx') {
 steps {
 sh '''
 set -e

 wget -nc http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz

 rm -rf nginx-${NGINX_VERSION}

 tar -xzf nginx-${NGINX_VERSION}.tar.gz

 cd nginx-${NGINX_VERSION}

 ./configure \
 --prefix=${NGINX_PATH} \
 --sbin-path=${NGINX_PATH}/bin/nginx \
 --conf-path=${NGINX_PATH}/conf/nginx.conf \
 --pid-path=${NGINX_PATH}/logs/nginx.pid \
 --error-log-path=${NGINX_PATH}/logs/error.log \
 --http-log-path=${NGINX_PATH}/logs/access.log \
 --with-http_ssl_module \
 --with-pcre

 make -j$(nproc)

 sudo make install
 '''
 }
 }

 stage('Restore Configuration') {
 steps {
 sh '''
 set -e

 echo "Restoring nginx.conf..."
 sudo cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/

 if [ -d ${BACKUP_PATH}/conf.d ]; then
 echo "Restoring conf.d..."
 sudo mkdir -p ${NGINX_PATH}/conf/conf.d
 sudo cp -r ${BACKUP_PATH}/conf.d/* ${NGINX_PATH}/conf/conf.d/ || true
 else
 echo "No conf.d backup found."
 fi

 if [ -d ${BACKUP_PATH}/ssl ]; then
 echo "Restoring SSL..."
 sudo mkdir -p ${NGINX_PATH}/ssl
 sudo cp -r ${BACKUP_PATH}/ssl/* ${NGINX_PATH}/ssl/ || true
 else
 echo "No SSL backup found."
 fi
 '''
 }
 }

 stage('Validate Configuration') {
 steps {
 sh '''
 set -e

 sudo ${NGINX_PATH}/bin/nginx -t \
 -c ${NGINX_PATH}/conf/nginx.conf
 '''
 }
 }

 stage('Restart Nginx') {
 steps {
 sh '''
 set -e

 sudo systemctl daemon-reload
 sudo systemctl restart nginx

 sudo systemctl status nginx --no-pager
 '''
 }
 }

 stage('Health Check') {
 steps {
 sh '''
 set -e

 echo "===== Nginx Version ====="
 ${NGINX_PATH}/bin/nginx -v

 echo "===== HTTP Check ====="
 curl -I http://localhost

 echo "===== Service Status ====="
 systemctl is-active nginx

 if systemctl is-active --quiet nginx; then
 echo "SUCCESS : Nginx is running."
 else
 echo "FAILED : Nginx service is not running."
 exit 1
 fi
 '''
 }
 }
 }

 post {

 success {
 emailext(
 subject: "Nginx ${NGINX_VERSION} Upgrade Successful",
 body: """
 Hello Team,

The Nginx upgrade completed successfully.

New Version : ${NGINX_VERSION}
 Installation : ${NGINX_PATH}

Tasks Completed:
 - Configuration backed up
 - Nginx compiled successfully
 - Configuration restored
 - Service restarted
 - Health check passed

Regards,
 Jenkins
 """,
 to: "ops-team@example.com"
 )
 }

 failure {
 sh '''
 set +e

 echo "====================================="
 echo "Upgrade failed. Starting rollback..."
 echo "====================================="

 wget -nc http://nginx.org/download/nginx-${OLD_VERSION}.tar.gz

 rm -rf nginx-${OLD_VERSION}

 tar -xzf nginx-${OLD_VERSION}.tar.gz

 cd nginx-${OLD_VERSION}

 ./configure \
 --prefix=${NGINX_PATH} \
 --sbin-path=${NGINX_PATH}/bin/nginx \
 --conf-path=${NGINX_PATH}/conf/nginx.conf \
 --pid-path=${NGINX_PATH}/logs/nginx.pid \
 --error-log-path=${NGINX_PATH}/logs/error.log \
 --http-log-path=${NGINX_PATH}/logs/access.log \
 --with-http_ssl_module \
 --with-pcre

 make -j$(nproc)

 sudo make install

 echo "Restoring nginx.conf..."
 sudo cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/

 if [ -d ${BACKUP_PATH}/conf.d ]; then
 sudo mkdir -p ${NGINX_PATH}/conf/conf.d
 sudo cp -r ${BACKUP_PATH}/conf.d/* ${NGINX_PATH}/conf/conf.d/ || true
 fi

 if [ -d ${BACKUP_PATH}/ssl ]; then
 sudo mkdir -p ${NGINX_PATH}/ssl
 sudo cp -r ${BACKUP_PATH}/ssl/* ${NGINX_PATH}/ssl/ || true
 fi

 sudo ${NGINX_PATH}/bin/nginx -t \
 -c ${NGINX_PATH}/conf/nginx.conf

 sudo systemctl restart nginx
 '''

 emailext(
 subject: "Nginx Upgrade Failed - Rollback Completed",
 body: """
 Hello Team,

The upgrade to Nginx ${NGINX_VERSION} failed.

Rollback has been completed successfully.

Current Version : ${OLD_VERSION}

Please review the Jenkins console log for additional details.

Regards,
 Jenkins
 """,
 to: "ops-team@example.com"
 )
 }

 always {
 cleanWs()
 }
 }
 }