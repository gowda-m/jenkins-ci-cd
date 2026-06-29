pipeline {
 agent any

 environment {
 NGINX_VERSION = "1.31.1"
 OLD_VERSION = "1.28.3"
 NGINX_PATH = "/opt/nginx"
 BACKUP_PATH = "/opt/backup"
 }

 stages {

 stage('Stop Existing Nginx (if running)') {
 steps {
 sh '''
 set -e

 if pgrep nginx > /dev/null; then
 echo "Nginx is running, stopping safely..."

 systemctl stop nginx || true
 pkill -f nginx || true

 sleep 2
 else
 echo "No nginx process running."
 fi
 '''
 }
 }

 stage('Backup Configurations') {
 steps {
 sh '''
 set -e

 mkdir -p ${BACKUP_PATH}

 if [ -f ${NGINX_PATH}/conf/nginx.conf ]; then
 cp ${NGINX_PATH}/conf/nginx.conf ${BACKUP_PATH}/
 fi

 mkdir -p ${BACKUP_PATH}/conf.d
 mkdir -p ${BACKUP_PATH}/ssl

 if [ -d ${NGINX_PATH}/conf/conf.d ]; then
 cp -r ${NGINX_PATH}/conf/conf.d/. ${BACKUP_PATH}/conf.d/ || true
 fi

 if [ -d ${NGINX_PATH}/ssl ]; then
 cp -r ${NGINX_PATH}/ssl/. ${BACKUP_PATH}/ssl/ || true
 fi
 '''
 }
 }

 stage('Download & Extract') {
 steps {
 sh '''
 set -e

 cd /opt

 if [ ! -f nginx-${NGINX_VERSION}.tar.gz ]; then
 curl -O http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
 fi

 rm -rf nginx-${NGINX_VERSION}
 tar -xzf nginx-${NGINX_VERSION}.tar.gz
 '''
 }
 }

 stage('Build & Install Nginx') {
 steps {
 sh '''
 set -e

 cd /opt/nginx-${NGINX_VERSION}

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
 make install
 '''
 }
 }

 stage('Restore Configuration') {
 steps {
 sh '''
 set -e

 mkdir -p ${NGINX_PATH}/conf/conf.d
 mkdir -p ${NGINX_PATH}/ssl

 cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/ || true

 cp -r ${BACKUP_PATH}/conf.d/. ${NGINX_PATH}/conf/conf.d/ || true
 cp -r ${BACKUP_PATH}/ssl/. ${NGINX_PATH}/ssl/ || true
 '''
 }
 }

 stage('Validate Configuration') {
 steps {
 sh '''
 set -e

 ${NGINX_PATH}/bin/nginx -t -c ${NGINX_PATH}/conf/nginx.conf
 '''
 }
 }

 stage('Start Nginx') {
 steps {
 sh '''
 set -e

 # Prefer systemd if available
 if systemctl list-unit-files | grep -q nginx; then
 systemctl start nginx
 else
 ${NGINX_PATH}/bin/nginx
 fi

 sleep 2
 '''
 }
 }

 stage('Health Check') {
 steps {
 sh '''
 set -e

 curl -I http://localhost || exit 1

 ${NGINX_PATH}/bin/nginx -v
 '''
 }
 }
 }

 post {

 success {
 echo "✅ Nginx upgraded successfully to ${NGINX_VERSION}"
 }

 failure {
 sh '''
 set +e

 echo "❌ Upgrade failed - starting rollback..."

 cd /opt/nginx-${OLD_VERSION} 2>/dev/null || true
 make install || true

 mkdir -p ${NGINX_PATH}/conf/conf.d
 mkdir -p ${NGINX_PATH}/ssl

 cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/ || true
 cp -r ${BACKUP_PATH}/conf.d/. ${NGINX_PATH}/conf/conf.d/ || true
 cp -r ${BACKUP_PATH}/ssl/. ${NGINX_PATH}/ssl/ || true

 ${NGINX_PATH}/bin/nginx -t || true
 systemctl restart nginx || true

 echo "Rollback completed."
 '''
 }
 }
 }