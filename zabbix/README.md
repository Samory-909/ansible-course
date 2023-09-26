#  Install Zabbix 6.5+
## Pre-requisite
```shell
cd
wget https://repo.zabbix.com/zabbix/6.5/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.5-1+ubuntu20.04_all.deb
sudo dpkg -i zabbix-release_6.5-1+ubuntu20.04_all.deb
sudo apt update
sudo apt install -y zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent
# Installation de postgresql
docker run -d --name db -e POSTGRES_PASSWORD=password  -v /opt/postgres:/var/lib/postgresql/data \
 -p 5432:5432 postgres:13.6
docker volume create portainer_data
docker run -d -p 32125:8000 -p 32126:9443 --name portainer --restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data portainer/portainer-ce:latest
```
## Creation de la base de donnees
```shell
# Connection a la base postgres
CREATE DATABASE zabbix;
CREATE ROLE zabbix WITH LOGIN ENCRYPTED PASSWORD 'zabbix';
```
## Peupler la base de donnees
```shell
cd /usr/share/zabbix-sql-scripts/postgresql

sudo -s
docker cp server.sql.gz db:/tmp/server.sql.gz
docker cp timescaledb.sql db:/tmp/timescaledb.sql

# dans le container db avec portainer
# 
su - postgres
# Connection a la base postgres
psql
CREATE DATABASE zabbix;
CREATE ROLE zabbix WITH LOGIN ENCRYPTED PASSWORD 'zabbix';
\q
#echo "CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;" | psql zabbix
exit
adduser zabbix 
su - zabbix 
cd /tmp
zcat server.sql.gz | psql zabbix 
#cat timescaledb.sql | psql zabbix
```

## Changer le password pour le daemon zabbix_server 
```shell
sudo vi /etc/zabbix/zabbix_server.conf
# change 
DBPassword=zabbix
```
## restart all processes
```shell
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```
## Installer le GUI en PHP
Completer l'installation avec un navigateur
```http://<ip>/zabbix```







