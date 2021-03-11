# mysql

## docker 进程
```
sudo docker run --name mysql-biut  \
--network intchain  -d --restart always  \
-e MYSQL_ROOT_PASSWORD=Sd0pL0KK \
-p 3306:3306 \
-v /home/docker/mysql_data/mysql-biut:/var/lib/mysql \
mysql:5.7.22

```

## game用户权限
```
create database game;
grant all privileges on game.* to 'game'@'%' identified by 'rfven2iMZf6rBJBi';
flush privileges; 

mysql -ugame -h18.166.72.237 -P3306 -prfven2iMZf6rBJBi
```

# redis

```
sudo docker run -d --name  redis --network intchain --restart always redis:3.2 --requirepass "Hj,NUj.PLqC+O-PGjF"
```


# stoneApp

```
sudo docker build -t biut .

sudo docker run --name biut \
--network intchain  -d --restart always \
--env NODE_ENV=pro \
-v /home/server/sec-sd:/home/nodeapp \
biut

log路径映射配置
systemctl stop docker
sudo cd /var/lib/docker/containers/35640f9881cb643cc20032e15a035a3a03924f540b28daf32cad7a89b336252c
vim config.v2.json
配置文件中添加路径映射(/root/docker/logs/biut:/root/.pm2/logs)，添加内容如下：
"/root/.pm2/logs":{"Source":"/root/docker/logs/biut","Destination":"/root/.pm2/logs","RW":true,"Name":"","Driver":"","Type":"bind","Propagation":"rprivate","Spec":{"Type":"bind","Source":"/root/docker/logs/biut","Target":"/root/.pm2/logs"},"SkipMountpointCreation":false}
systemctl start docker
```


# 后台管理

```
sudo docker build -t biut-admin .

sudo docker run --name biut-admin \
--network intchain  -d --restart always \
--env NODE_ENV=pro \
-v /home/server/sd-sec-admin-server:/home/nodeapp \
biut-admin

```

# nginx

```
docker run -d \
  -p 80:80 \
  --name nginx \
  --network intchain \
  -p 443:443 \
  -v /home/web:/usr/share/nginx/html \
  -v /home/docker/nginx/nginx.conf:/etc/nginx/nginx.conf \
  -v /home/docker/nginx/conf.d:/etc/nginx/conf.d \
  -v /home/docker/nginx/log:/var/log/nginx \
  -v /home/docker/nginx/cert:/cert \
  nginx

  ```
