
#### nginx 启动
```docker
docker run -p 80:80 --name nginx  \
-v /home/docker/nginx/www:/www -v \
/home/docker/nginx/config/conf.d:/etc/nginx/nginx.conf --privileged=true \
-v /home/docker/nginx/logs:/www/logs \
-d nginx;
```

```docker
docker run -it -p 80:80 --name nginx \
--restart=on-failure:10 \
-v /usr/local/hexo/public:/usr/share/nginx/html \
-v /home/docker/nginx/config/conf.d/nginx.conf:/etc/nginx/nginx.conf \
-v /home/docker/nginx/logs:/www/logs \
--privileged=true  -d nginx;
```

### redis 启动
```docker
docker run -it -d -p 6379:6379 --name redis \
-v /home/docker/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf 
-v /home/docker/redis/data:/data \
-d redis;
```


### mySql 启动
```docker
docker run --name mysql -p 3306:3306 \
-v /home/docker/mysql:/var/lib/mysql  \
--restart=always -e MYSQL_ROOT_PASSWORD=Zm19930607 -d mysql;

```
```docker
docker run  --name mysql -p 3306:3306  restart: always \
-v /home/docker/mysql/data:/var/lib/mysql \
-e MYSQL_USER="stupidzhang" -e MYSQL_PASSWORD="Zm19930607" -e MYSQL_ROOT_PASSWORD="Zm19930607" \
-d mysql:5.7 --character-set-server=utf8 --collation-server=utf8_general_ci;
```



### docker 重启
```docker
systemctl restart docker;
```
#### 清理所有停止运行的容器：
```docker

docker container prune
# or
docker rm $(docker ps -aq)
```
#### 清理所有悬挂
```docker

docker image prune
# or
docker rmi $(docker images -qf "dangling=true")
```
```docker
#清理所有无用数据卷
docker volume prune

```


### b3log
```docker
docker run --detach --name solo --network=host \
    --env RUNTIME_DB="MYSQL" \
    --env JDBC_USERNAME="root" \
    --env JDBC_PASSWORD="Zm19930607" \
    --env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
    --env JDBC_URL="jdbc:mysql://118.31.61.143:3306/solo?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
    b3log/solo --listen_port=80 --server_scheme=http --server_host=solo.stupidzhang.com --server_port=


作者：88250
链接：https://hacpai.com/article/1492881378588
来源：黑客派
协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/

```


```docker
docker run -d --hostname localhost --name rabbitmq -p 15672:15672 -p 5672:5672 \
-v /home/docker/rabbitmq/data/:/var/rabbitmq/lib   \
rabbitmq:management;
```

docker stop $(docker ps --filter name="sentinel" -aq);

docker rm $(docker ps --filter name="sentinel" -aq);

docker stop $(docker ps --filter name="redis" -aq);

docker rm $(docker ps --filter name="redis" -aq);


docker stop $(docker ps --filter name="rabbit" -aq);

docker rm $(docker ps --filter name="rabbit" -aq);
