# gitea

Installing Gitea – A self-hosted Git Server on Ubuntu 20.04 LTS
```
sudo apt update
sudo apt install wget -y
sudo apt install git -y

sudo apt install mysql-server mysql-client -y
sudo mysql -u root -p
mysql> CREATE USER 'gitea' IDENTIFIED BY 'secret';
mysql> CREATE DATABASE gitea CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_unicode_ci';
mysql> GRANT ALL PRIVILEGES ON gitea.* TO 'gitea';
mysql> FLUSH PRIVILEGES;
mysql> exit

sudo wget -O /usr/local/bin/gitea https://dl.gitea.io/gitea/1.11.4/gitea-1.11.4-linux-amd64
sudo chmod +x /usr/local/bin/gitea
gitea --version

sudo adduser --system --shell /bin/bash --gecos 'Git Version Control' --group --disabled-password --home /home/git git

sudo mkdir -pv /var/lib/gitea/{custom,data,log}
sudo chown -Rv git:git /var/lib/gitea
sudo chmod -Rv 750 /var/lib/gitea
sudo mkdir -v /etc/gitea
sudo chown -Rv root:git /etc/gitea
sudo chmod -Rv 770 /etc/gitea

sudo nano /etc/systemd/system/gitea.service

copy bellow code and save:
> [Unit]
> Description=Gitea (Git with a cup of tea)
> After=syslog.target
> After=network.target
> Requires=mysql.service
> 
> [Service]
> LimitMEMLOCK=infinity
> LimitNOFILE=65535
> RestartSec=2s
> Type=simple
> User=git
> Group=git
> WorkingDirectory=/var/lib/gitea/
> ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
> Restart=always
> Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea
> CapabilityBoundingSet=CAP_NET_BIND_SERVICE
> AmbientCapabilities=CAP_NET_BIND_SERVICE
> 
> [Install]
> WantedBy=multi-user.target

sudo systemctl start gitea
sudo systemctl status gitea
sudo systemctl enable gitea

ip a
http://ip_address:3000
```
<a href="https://imgur.com/8FvATFE"><img src="https://i.imgur.com/8FvATFE.png" title="source: imgur.com" /></a><br/><br/>

How to Install Gitea on CentOS 7
```
yum update
yum install wget git nano epel-release

useradd git

mkdir -p /etc/gitea /var/lib/gitea/{custom,data,indexers,public,log}
chown git:git /var/lib/gitea/{data,indexers,log}
chmod 750 /var/lib/gitea/{data,indexers,log}
chown root:git /etc/gitea
chmod 770 /etc/gitea

yum install mariadb-server
systemctl enable mariadb
systemctl start mariadb
mysql_secure_installation

mysql -u root -p
mysql> create database gitea;
mysql> grant all on gitea.* to gitea@localhost identified by 'new_pass';
mysql> flush privileges;
mysql> quit

export GITEAVER=1.8.3
wget https://github.com/go-gitea/gitea/releases/download/v${GITEAVER}/gitea-${GITEAVER}-linux-amd64 -O /usr/local/bin/gitea
chmod +x /usr/local/bin/gitea
gitea -v

nano /etc/systemd/system/gitea.service
copy & pase, save bellow code:

[Unit]
Description=Gitea
After=syslog.target
After=network.target
After=mariadb.service

[Service]
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
ExecStart=/usr/local/bin/gitea web -c /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea

[Install]
WantedBy=multi-user.target

systemctl daemon-reload
systemctl start gitea
systemctl status gitea

yum install nginx
nano /etc/nginx/conf.d/gitea.conf
copy & paste, save bellow code:

upstream gitea {
     server 127.0.0.1:3000
}

server {
     listen 80;
     server_name git.example.com;

     location / {
        proxy_pass http://gitea;
     }
}

systemctl restart nginx

http://git.example.com/install
```


##### Reference
official docs: https://docs.gitea.io/en-us/install-from-binary/#installation-from-binary

ubuntu: https://linuxhint.com/install_gitea_ubuntu_self_hosted_git/

CentOS/AWS AMI 2: https://www.linuxcloudvps.com/blog/how-to-install-gitea-on-centos-7/ 
