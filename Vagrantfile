# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.vm.box = "sbeliakou-vagrant-centos-7.3-x86_64-minimal.box"
    config.vm.hostname = "jenkins"
    config.vm.network "private_network", ip: "192.168.50.10"
    config.vm.synced_folder "master/", "/opt/jenkins/master", mount_options: ['dmode=777', 'fmode=777']
    config.vm.synced_folder "maven/", "/opt/maven", mount_options: ['dmode=777', 'fmode=777']
    config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"
    vb.name = "jenkins"
  end
  config.vm.provision "shell", inline: <<-SHELL

yum -y install nginx
rm -rf /etc/nginx/nginx.conf

cat > /etc/nginx/nginx.conf << EOF

user nginx;
worker_processes 1;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    access_log  /var/log/nginx/access.log;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
    ignore_invalid_headers off;
server {
        listen       80 default_server;
        server_name  jenkins;
        root         /usr/share/nginx/html;
      
location / {
           proxy_pass http://localhost:8080;
           proxy_read_timeout  90;
                      }   
     
error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;      
        }
}
}

EOF

yum -y install java

cd /opt/
wget http://ftp.byfly.by/pub/apache.org/maven/maven-3/3.5.0/binaries/apache-maven-3.5.0-bin.tar.gz
tar -xvf apache-maven-3.5.0-bin.tar.gz

group add jenkins
useradd jenkins -d /opt/jenkins
mkdir -p /opt/jenkins/bin
cp /vagrant/jenkins.war /opt/jenkins/bin/

chown -R jenkins:jenkins /opt/jenkins
chmod -R 775 /opt/jenkins


cat > /etc/systemd/system/multi-user.target.wants/jenkins.service << EOF
[Unit]
Description=Jenkins Service
After=network.target

[Service]

Type=simple
User=jenkins
Group=jenkins
Environment=JENKINS_HOME=/opt/jenkins/master
Environment=JENKINS_DIR=/opt/jenkins/bin
ExecStart=/usr/bin/java -jar /opt/jenkins/bin/jenkins.war
Restart=on-abort

[Install]
WantedBy=multi-user.target

EOF

systemctl daemon-reload 
systemctl enable nginx
systemctl enable jenkins
systemctl start nginx
systemctl start jenkins 

echo "Please copy the password from either location and paste it in browser: " 
cat /opt/jenkins/master/secrets/initialAdminPassword

SHELL

end

