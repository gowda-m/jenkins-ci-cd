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

 mkdir -p ${BACKUP_PATH}

 cp ${NGINX_PATH}/conf/nginx.conf ${BACKUP_PATH}/
 cp -r ${NGINX_PATH}/conf/conf.d ${BACKUP_PATH}/
 cp -r ${NGINX_PATH}/ssl ${BACKUP_PATH}/
 '''
 }
 }

 stage('Download & Build Nginx') {
 steps {
 sh '''
 set -e

 #if [ ! -f nginx-${NGINX_VERSION}.tar.gz ]; then
 #wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
 #fi

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

 cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/

 mkdir -p ${NGINX_PATH}/conf/conf.d
 cp -r ${BACKUP_PATH}/conf.d/* ${NGINX_PATH}/conf/conf.d/

 mkdir -p ${NGINX_PATH}/ssl
 cp -r ${BACKUP_PATH}/ssl/* ${NGINX_PATH}/ssl/
 '''
 }
 }

 stage('Validate Configuration') {
 steps {
 sh '''
 set -e

 ${NGINX_PATH}/bin/nginx -t \
 -c ${NGINX_PATH}/conf/nginx.conf
 '''
 }
 }

 stage('Restart Nginx') {
 steps {
 sh '''
 set -e

 systemctl daemon-reload
 systemctl restart nginx
 systemctl status nginx --no-pager
 '''
 }
 }

 stage('Health Check') {
 steps {
 sh '''
 set -e

 curl -I http://localhost

 if systemctl is-active --quiet nginx; then
 echo "Nginx is running."
 else
 exit 1
 fi
 '''
 }
 }
 }

 post {

 success {
 emailext(
 subject: "Nginx Upgrade Successful",
 body: """
 Nginx has been upgraded successfully.

Version : ${NGINX_VERSION}
 Installation Path : ${NGINX_PATH}

Configuration restored successfully.
 Service is running normally.
 """,
 to: "ops-team@example.com"
 )
 }

 failure {
 sh '''
 set +e

 echo "Upgrade failed. Starting rollback..."

 wget -nc http://nginx.org/download/nginx-${OLD_VERSION}.tar.gz

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

 cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/
 cp -r ${BACKUP_PATH}/conf.d/* ${NGINX_PATH}/conf/conf.d/
 cp -r ${BACKUP_PATH}/ssl/* ${NGINX_PATH}/ssl/

 ${NGINX_PATH}/bin/nginx -t \
 -c ${NGINX_PATH}/conf/nginx.conf

 systemctl restart nginx
 '''

 emailext(
 subject: "Nginx Upgrade Failed - Rollback Completed",
 body: """
 Upgrade to Nginx ${NGINX_VERSION} failed.

Rollback completed successfully.

Current Version : ${OLD_VERSION}

Please review the Jenkins build logs.
 """,
 to: "ops-team@example.com"
 )
 }
 }
 }