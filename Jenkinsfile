pipeline {
 agent any

 environment {
 NGINX_VERSION = "1.31.1"
 OLD_VERSION = "1.28.3"
 NGINX_PATH = "/opt/nginx"
 BACKUP_PATH = "/opt/backup"
 }

 stages {

 stage('Debug') {
 steps {
 sh '''
 whoami
 id
 sudo -n whoami
 '''
 }
 }

 stage('Stop Nginx') {
 steps {
 sh '''
 set -e

 if sudo -n systemctl is-active --quiet nginx; then
 echo "Stopping nginx..."
 sudo -n systemctl stop nginx
 else
 echo "Nginx already stopped."
 fi
 '''
 }
 }

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

 stage('Extract Source') {
 steps {
 sh '''
 set -e

 cd /opt

 test -f nginx-${NGINX_VERSION}.tar.gz

 rm -rf nginx-${NGINX_VERSION}

 tar -xzf nginx-${NGINX_VERSION}.tar.gz
 '''
 }
 }

 stage('Compile Nginx') {
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

 sudo -n make install
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

 stage('Validate Configuration') {
 steps {
 sh '''
 set -ex

 sudo -n ${NGINX_PATH}/bin/nginx \
 -t \
 -c ${NGINX_PATH}/conf/nginx.conf
 '''
 }
 }

 stage('Start Nginx') {
 steps {
 sh '''
 set -e

 sudo -n systemctl start nginx

 sleep 3

 sudo -n systemctl status nginx --no-pager
 '''
 }
 }

 stage('Health Check') {
 steps {
 sh '''
 set -e

 curl -I http://localhost

 sudo -n ${NGINX_PATH}/bin/nginx -v
 '''
 }
 }
 }

 post {

 success {
 echo "========================================"
 echo "Nginx upgraded successfully."
 echo "Version : ${NGINX_VERSION}"
 echo "========================================"
 }

 failure {

 sh '''
 set +e

 echo "Upgrade failed. Rolling back..."

 sudo -n systemctl stop nginx

 if [ -d /opt/nginx-${OLD_VERSION} ]; then
 cd /opt/nginx-${OLD_VERSION}
 sudo -n make install
 fi

 sudo mkdir -p ${NGINX_PATH}/conf/conf.d
 sudo mkdir -p ${NGINX_PATH}/ssl

 [ -f ${BACKUP_PATH}/nginx.conf ] && \
 sudo cp ${BACKUP_PATH}/nginx.conf ${NGINX_PATH}/conf/

 [ -d ${BACKUP_PATH}/conf.d ] && \
 sudo cp -a ${BACKUP_PATH}/conf.d/. ${NGINX_PATH}/conf/conf.d/

 [ -d ${BACKUP_PATH}/ssl ] && \
 sudo cp -a ${BACKUP_PATH}/ssl/. ${NGINX_PATH}/ssl/

 sudo -n systemctl start nginx

 sudo -n systemctl status nginx --no-pager
 '''

 echo "Rollback completed."
 }
 }
 }