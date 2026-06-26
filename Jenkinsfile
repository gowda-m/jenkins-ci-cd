pipeline {
 agent any

 environment {
 NGINX_VERSION = "1.31.1"
 TARBALL = "/opt/nginx/nginx-${NGINX_VERSION}.tar.gz"
 NGINX_BIN = "/usr/local/nginx/sbin/nginx"
 PREFIX = "/usr/local/nginx"
 }

 stages {

 stage('Backup Config') {
 steps {
 sh '''
 echo "Backing up nginx config..."

 sudo mkdir -p /opt/backup
 sudo cp -a ${PREFIX} /opt/backup/nginx_backup 2>/dev/null || true

 echo "Backup done"
 '''
 }
 }

 stage('Build & Install Nginx') {
 steps {
 sh '''
 echo "Starting build..."

 cd /tmp
 rm -rf nginx-${NGINX_VERSION}

 tar -xzf ${TARBALL}
 cd nginx-${NGINX_VERSION}

 ./configure \
 --prefix=${PREFIX} \
 --sbin-path=${NGINX_BIN} \
 --conf-path=${PREFIX}/conf/nginx.conf \
 --with-http_ssl_module

 make
 sudo make install

 echo "Build completed"
 '''
 }
 }

 stage('Validate & Reload') {
 steps {
 sh '''
 echo "Testing config..."

 sudo ${NGINX_BIN} -t

 echo "Reloading nginx..."
 sudo ${NGINX_BIN} -s reload

 echo "Version:"
 ${NGINX_BIN} -v
 '''
 }
 }
 }

 post {
 success {
 emailext(
 to: "gowda.m@intelizign.com",
 subject: "SUCCESS: Nginx ${NGINX_VERSION} Deployed",
 body: "Nginx ${NGINX_VERSION} built and deployed successfully."
 )
 }

 failure {
 sh '''
 echo "Rollback started..."

 sudo rm -rf ${PREFIX}
 sudo cp -a /opt/backup/nginx_backup ${PREFIX}

 sudo ${NGINX_BIN} -s reload || true

 echo "Rollback completed"
 '''

 emailext(
 to: "gowda.m@intelizign.com",
 subject: "FAILED: Nginx Rollback Executed",
 body: "Nginx deployment failed and rollback was performed."
 )
 }
 }
 }