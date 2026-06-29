pipeline {
 agent any

 environment {
 NGINX_VERSION = "1.31.1"
 OLD_VERSION = "1.28.3"
 NGINX_PATH = "/opt/nginx"
 BACKUP_PATH = "/opt/backup"
 }

 stages {

 stage('Backup Configuration') {
 steps {
 sh '''
 set -e

 sudo mkdir -p ${BACKUP_PATH}

 [ -f ${NGINX_PATH}/conf/nginx.conf ] && \
 sudo cp ${NGINX_PATH}/conf/nginx.conf ${BACKUP_PATH}/

 sudo mkdir -p ${BACKUP_PATH}/conf.d
 sudo mkdir -p ${BACKUP_PATH}/ssl

 [ -d ${NGINX_PATH}/conf/conf.d ] && \
 sudo cp -a ${NGINX_PATH}/conf/conf.d/. ${BACKUP_PATH}/conf.d/

 [ -d ${NGINX_PATH}/ssl ] && \
 sudo cp -a ${NGINX_PATH}/ssl/. ${BACKUP_PATH}/ssl/
 '''
 }
 }

 stage('Stop Nginx') {
 steps {
 sh '''
 set +e

 if systemctl is-active --quiet nginx; then
 echo "Stopping nginx..."
 sudo systemctl stop nginx
 sleep 3
 fi

 sudo pkill -9 nginx 2>/dev/null || true
 '''
 }
 }

 stage('Extract Source') {
 steps {
 sh '''
 set -e

 cd /opt

 if [ ! -f nginx-${NGINX_VERSION}.tar.gz ]; then
 curl -LO https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
 fi

 sudo rm -rf nginx-${NGINX_VERSION}

 tar -xzf nginx-${NGINX_VERSION}.tar.gz
 '''
 }
 }

 stage('Configure') {
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
 '''
 }
 }

 stage('Compile') {
 steps {
 sh '''
 set -e

 cd /opt/nginx-${NGINX_VERSION}

 make -j$(nproc)
 '''
 }
 }

 stage('Install') {
 steps {
 sh '''
 set -e

 cd /opt/nginx-${NGINX_VERSION}

 sudo make install
 '''
 }
 }

 stage('Restore Configuration') {
 steps {
 sh '''
 set -e

 sudo mkdir -p ${NGINX_PATH}/conf/conf.d
 sudo mkdir -p ${NGINX_PATH}/ssl

 [ -f ${BACKUP_PATH}/nginx.conf ] && \
 sudo cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/

 [ -d ${BACKUP_PATH}/conf.d ] && \
 sudo cp -a ${BACKUP_PATH}/conf.d/. ${NGINX_PATH}/conf/conf.d/

 [ -d ${BACKUP_PATH}/ssl ] && \
 sudo cp -a ${BACKUP_PATH}/ssl/. ${NGINX_PATH}/ssl/
 '''
 }
 }

 stage('Validate') {
 steps {
 sh '''
 set -e

 ${NGINX_PATH}/bin/nginx -t \
 -c ${NGINX_PATH}/conf/nginx.conf
 '''
 }
 }

 stage('Start Nginx') {
 steps {
 sh '''
 set -e

 sudo systemctl start nginx

 sleep 5
 '''
 }
 }

 stage('Health Check') {
 steps {
 sh '''
 set -e

 curl -I http://localhost

 ${NGINX_PATH}/bin/nginx -v

 systemctl is-active --quiet nginx
 '''
 }
 }
 }

 post {

 success {
 sh '''
 echo "Upgrade Successful"

 ${NGINX_PATH}/bin/nginx -v
 '''
 }

 failure {
 sh '''
 set +e

 echo "Upgrade failed. Rolling back..."

 sudo systemctl stop nginx || true

 if [ -d /opt/nginx-${OLD_VERSION} ]; then
 cd /opt/nginx-${OLD_VERSION}

 sudo make install
 fi

 sudo mkdir -p ${NGINX_PATH}/conf/conf.d
 sudo mkdir -p ${NGINX_PATH}/ssl

 [ -f ${BACKUP_PATH}/nginx.conf ] && \
 sudo cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/

 [ -d ${BACKUP_PATH}/conf.d ] && \
 sudo cp -a ${BACKUP_PATH}/conf.d/. ${NGINX_PATH}/conf/conf.d/

 [ -d ${BACKUP_PATH}/ssl ] && \
 sudo cp -a ${BACKUP_PATH}/ssl/. ${NGINX_PATH}/ssl/

 ${NGINX_PATH}/bin/nginx -t || true

 sudo systemctl start nginx || true

 echo "Rollback completed."
 '''
 }
 }
 }