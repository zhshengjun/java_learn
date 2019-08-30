
#### nginx 启动
```docker
docker run -p 80:80 --name nginx  \
-v /home/docker/nginx/www:/www -v \
/home/docker/nginx/conf.d/nginx.conf:/etc/nginx/nginx.conf \
--privileged=true \
-v /home/docker/nginx/logs:/www/logs \
-d nginx;
```
--rm
```docker
docker run  -p 80:80 -p 443:443 --name nginx \
--restart=on-failure:10 \
-v /usr/local/hexo/public:/usr/share/nginx/html \
-v /home/docker/nginx/conf.d/nginx.conf:/etc/nginx/nginx.conf \
-v /home/docker/nginx/logs:/www/logs \
--privileged=true  -d nginx;
```
```docker
docker run  -p 80:80 --name nginx \
--restart=on-failure:10 \
-v /usr/local/hexo/public:/usr/share/nginx/html \
-v /home/docker/nginx/conf.d/nginx.conf:/etc/nginx/nginx.conf \
-v /home/docker/nginx/logs:/www/logs \
--privileged=true  -d nginx;

```docker
docker run -d  -p 80:80 -p 443:443 --name nginx \
-v /home/docker/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /home/docker/nginx/conf.d:/etc/nginx/conf.d \
-v /home/docker/nginx/logs:/var/log/nginx \
nginx


docker run -d -p 80:80 -p 443:443 --name nginx \
-v /home/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/docker/nginx/conf.d:/etc/nginx/conf.d \
-v /home/docker/nginx/www:/usr/share/nginx/html \
-v /home/docker/nginx/logs:/var/log/nginx \
-v /home/docker/nginx/certs:/ssl/ \
nginx
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
docker run --name mysql2 -p 3308:3308 \
--restart=always -e MYSQL_ROOT_PASSWORD=Zm19930607 -d mysql;
-v /home/docker/mysql:/var/lib/mysql  \
--restart=always -e MYSQL_ROOT_PASSWORD=Zm19930607 -d mysql;

```
```docker
docker run  --name mysql -p 3306:3306  --restart=always \
-v /home/docker/mysql/data:/var/lib/mysql \
-e MYSQL_USER="stupidzhang" -e MYSQL_PASSWORD="Zm19930607" -e MYSQL_ROOT_PASSWORD="Zm19930607" \
-d mysql --character-set-server=utf8 --collation-server=utf8_general_ci;
```

docker run  --name mysql2 -p 3308:3308   --restart=always \
-v /home/docker/mysql/data:/var/lib/mysql \
-e MYSQL_USER="stupidzhang" -e MYSQL_PASSWORD="Zm19930607" -e MYSQL_ROOT_PASSWORD="Zm19930607" \
-d mysql --character-set-server=utf8 --collation-server=utf8_general_ci;





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
docker run -d --name solo2 --network=host  \
    --env RUNTIME_DB="MYSQL" \
    --env JDBC_USERNAME="root" \
    --env JDBC_PASSWORD="Zm19930607" \
    --env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
    --env JDBC_URL="jdbc:mysql://118.31.61.143:3308/solo?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
    b3log/solo --listen_port=8081 --server_scheme=http --server_host=localhost --server_port=

docker run -d --name solo --network=host  \
    --env RUNTIME_DB="MYSQL" \
    --env JDBC_USERNAME="root" \
    --env JDBC_PASSWORD="Zm19930607" \
    --env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
    --env JDBC_URL="jdbc:mysql://118.31.61.143:3306/solo?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
    b3log/solo --listen_port=8082 --server_scheme=http --server_host=solo.stupidzhang.com  --server_port=

docker run --detach --name solo --network=host \
    --env RUNTIME_DB="MYSQL" \
    --env JDBC_USERNAME="root" \
    --env JDBC_PASSWORD="Zm19930607" \
    --env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
    --env JDBC_URL="jdbc:mysql://118.31.61.143:3306/solo?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
    b3log/solo --listen_port=8082 --server_scheme=http --server_host=solo.stupidzhang.com --server_port=






docker run --detach --name pipe --network=host \
b3log/pipe --mysql="root:Zm19930607@(118.31.61.143:3306)/pipe?charset=utf8mb4&parseTime=True&loc=Local" \
--runtime_mode=prod --port=8083 \
--server=http://pipe.stupidzhang.com


docker run -d --name pipe --network=host  \
    --env RUNTIME_DB="MYSQL" \
    --env JDBC_USERNAME="root" \
    --env JDBC_PASSWORD="Zm19930607" \
    --env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
    --env JDBC_URL="jdbc:mysql://118.31.61.143:3306/pipe?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
    b3log/pipe --listen_port=8083 --server_scheme=http --server_host=pipe.stupidzhang.com  --server_port=

```


```docker
docker run -d --hostname localhost --name rabbitmq -p 15672:15672 -p 5672:5672 \
-v /home/docker/rabbitmq/data/:/var/rabbitmq/lib   \
rabbitmq:management;
```

docker stop $(docker ps --filter name="sentinel" -aq);

docker rm $(docker ps --filter name="sentinel" -aq);



sudo docker pull ruibaby/halo

docker run --rm -it -d --name halo -p 8090:8090  -v ~/home/docker/halo:/root/.halo ruibaby/halo

docker stop $(docker ps --filter name="redis" -aq);

docker rm $(docker ps --filter name="redis" -aq);


docker stop $(docker ps --filter name="rabbit" -aq);

docker rm $(docker ps --filter name="rabbit" -aq);

```docker

docker run --detach --name pipe --network=host \
b3log/pipe --mysql="root:Zm19930607@(127.0.0.1:3306)/pipe?charset=utf8mb4&parseTime=True&loc=Local" \
--runtime_mode=prod --port=5897 --server=http://localhost:5897

docker run --detach --name pipe --network=host \
--env RUNTIME_DB="MYSQL" \
    --env JDBC_USERNAME="root" \
    --env JDBC_PASSWORD="Zm19930607" \
    --env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
    --env JDBC_URL="jdbc:mysql://118.31.61.143:3306/solo?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
b3log/pipe  --runtime_mode=prod --port=5897 --server=http://localhost:5897



docker run --detach --name solo --network=host \
    --volume /home/docker/solo/skins/:/opt/solo/skins/
    --env RUNTIME_DB="MYSQL" \
    --env JDBC_USERNAME="root" \
    --env JDBC_PASSWORD="Zm19930607" \
    --env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
    --env JDBC_URL="jdbc:mysql://127.0.0.1:3306/solo?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
    b3log/solo --listen_port=8082 --server_scheme=http --server_host=solo.stupidzhang.com --server_port=


```


```docker
docker run --env MODE=standalone \
--restart=always \
--env SPRING_DATASOURCE_PLATFORM=mysql \
--env MYSQL_MASTER_SERVICE_DB_NAME=nacos_config \
--env MYSQL_MASTER_SERVICE_HOST=118.31.61.143 \
--env MYSQL_MASTER_SERVICE_USER=root \
--env MYSQL_MASTER_SERVICE_PASSWORD=Zm19930607 \
--env MYSQL_SLAVE_SERVICE_HOST=118.31.61.143 \
--env MYSQL_MASTER_SERVICE_DB_NAME=nacos \
--name nacos -d -p 8848:8848 nacos/nacos-server

```

```docker

docker run -d \
--name nacos \
-e PREFER_HOST_MODE=118.31.61.143 \
-e MODE=standalone \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_MASTER_SERVICE_HOST=118.31.61.143 \
-e MYSQL_MASTER_SERVICE_PORT=3306 \
-e MYSQL_MASTER_SERVICE_USER=root \
-e MYSQL_MASTER_SERVICE_PASSWORD=Zm19930607 \
-e MYSQL_MASTER_SERVICE_DB_NAME=nacos_config \
-p 8848:8848 \
nacos/nacos-server;

```

docker run --env MODE=standalone --name nacos -d  -p 8848:8848  paderlol/nacos
wget https://raw.githubusercontent.com/zq2599/blog_demos/master/nacosdemo/dockerfiles/simple/docker-compose.yml && \
docker-compose up --scale provider=6 -d
docker run --env MODE=standalone --name nacos -d -p 8848:8848 nacos/nacos-server

docker run --rm -it -d --name halo -p 8090:8090 -v /home/docker/halo:/root/.halo ruibaby/halo;
docker run  -d --name halo -p 8090:8090 -v /home/docker/halo:/root/.halo ruibaby/halo;


docker run --detach --name solo --network=host \
--env RUNTIME_DB="MYSQL" \
--env JDBC_USERNAME="root" \
--env JDBC_PASSWORD="Zm19930607" \
--env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
--env JDBC_URL="jdbc:mysql://118.31.61.143:3306/solo?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
--volume /home/docker/solo/skins/:/opt/solo/skins/  \
b3log/solo --listen_port=8082 --server_scheme=http --server_host=solo.stupidzhang.com

--volume /home/docker/solo/skins/:/opt/solo/skins/ \


docker cp nginx:/etc/nginx/nginx.conf /home/docker/nginx/conf/nginx.conf

docker cp nginx:/etc/nginx/conf.d /home/docker/nginx/conf/conf.d


docker run -d -p 80:80 -p 443:443 --name nginx \
-v /home/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/docker/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/docker/nginx/www:/usr/share/nginx/html \
-v /home/docker/nginx/ssl:/ssl/ \
-v /home/docker/nginx/logs:/var/log/nginx \
nginx

docker run -d -p 80:80 -p 443:443 --name nginx \
-v /dockerData/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /dockerData/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/docker/nginx/ssl:/ssl/ \
-v /dockerData/nginx/www:/usr/share/nginx/html \
-v /dockerData/nginx/logs:/var/log/nginx \
nginx 


docker run --detach --name pipe --network=host \
b3log/pipe --mysql="root:Zm19930607@(127.0.0.1:3306)/pipe?charset=utf8mb4&parseTime=True&loc=Local" \
--runtime_mode=prod --port=8088 --server=http://pipe.stupidzhang.com


docker run --detach --name pipe --network=host \
b3log/pipe --mysql="root:Zm19930607@(127.0.0.1:3306)/pipe?charset=utf8mb4&parseTime=True&loc=Local" \
--runtime_mode=prod --port=5897 --server=http://localhost:5897


