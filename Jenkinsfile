pipeline {
 agent any

 environment {
 NGINX_VERSION = "1.31.1"
 OLD_VERSION = "1.28.3"
 NGINX_PATH = "/opt/nginx"
 BACKUP_PATH = "/opt/backup"
 }

 stages {

 stage('Backup Configurations') {
 steps {
 sh '''
 set -e
 mkdir -p ${BACKUP_PATH}

 if [ -f ${NGINX_PATH}/conf/nginx.conf ]; then
 cp ${NGINX_PATH}/conf/nginx.conf ${BACKUP_PATH}/
 fi

 if [ -d ${NGINX_PATH}/conf/conf.d ]; then
 cp -r ${NGINX_PATH}/conf/conf.d ${BACKUP_PATH}/
 fi

 if [ -d ${NGINX_PATH}/ssl ]; then
 cp -r ${NGINX_PATH}/ssl ${BACKUP_PATH}/
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

 stage('Build & Install') {
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

 stage('Restore Configurations') {
 steps {
 sh '''
 set -e

 cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/ || true

 mkdir -p ${NGINX_PATH}/conf/conf.d
 cp -r ${BACKUP_PATH}/conf.d/* ${NGINX_PATH}/conf/conf.d/ || true

 mkdir -p ${NGINX_PATH}/ssl
 cp -r ${BACKUP_PATH}/ssl/* ${NGINX_PATH}/ssl/ || true
 '''
 }
 }

 stage('Validate Config') {
 steps {
 sh '''
 set -e
 ${NGINX_PATH}/bin/nginx -t -c ${NGINX_PATH}/conf/nginx.conf
 '''
 }
 }

 stage('Restart Nginx') {
 steps {
 sh '''
 set -e

 if systemctl list-units --type=service | grep -q nginx; then
 systemctl restart nginx
 else
 ${NGINX_PATH}/bin/nginx -s reload || ${NGINX_PATH}/bin/nginx
 fi
 '''
 }
 }

 stage('Health Check') {
 steps {
 sh '''
 set -e

 sleep 3
 curl -I http://localhost || exit 1

 ${NGINX_PATH}/bin/nginx -v
 '''
 }
 }
 }

 post {

 success {
 echo "Nginx upgrade successful to ${NGINX_VERSION}"
 }

 failure {
 sh '''
 set +e
 echo "Upgrade failed - rolling back..."

 if [ -d /opt/nginx-${OLD_VERSION} ]; then
 cd /opt/nginx-${OLD_VERSION}
 make install
 fi

 cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/ || true
 cp -r ${BACKUP_PATH}/conf.d/* ${NGINX_PATH}/conf/conf.d/ || true
 cp -r ${BACKUP_PATH}/ssl/* ${NGINX_PATH}/ssl/ || true

 ${NGINX_PATH}/bin/nginx -t && ${NGINX_PATH}/bin/nginx -s reload || true
 '''

 echo "Rollback completed to ${OLD_VERSION}"
 }
 }
 }