# docker开发环境

------

为了方便开启所有开发需要的服务，也能保证环境一致。通过docker compose编排工具整合了一下官方常用镜像。里面包括的服务：
> * mysql5、mysql8
> * mongo、mongo-express
> * redis
> * rabbitmq
> * postgres
> * mssql
> * php-fpm5、php-fpm7、php-fpm8
> * php-cli5、php-cli7、php-cli8
> * nginx
> * java
------
用到的[dockerfile](https://github.com/wms3001/dockerFile.git)文件。
## 使用的docker命令
```
启动所有容器
docker compose up
后台启动所有容器
docker compose up -d
停止所有容器
docker compose stop
删除所有容器
docker  compose rm
```
## .env
容器使用的镜像版本统一从这个文件配置。
```
MYSQL_VERSION=8.0.31
MYSQL_VERSION_OLD=5.7.40
MONGO_VERSION=6.0.2
MONGO_EXPRESS_VERSION=latest
REDIS_VERSION=7.0.5
RABBITMQ_VERSION=management
POSTGRES_VERSION=15.0
MSSQL_VERSION=2022-latest
PHPFPM5_VERSION=5.5-fpm
PHPFPM7_VERSION=7.4-fpm
PHPFPM8_VERSION=8.1-fpm
```
## 容器说明
### 1. mysql5
```
镜像
image: mysql:${MYSQL_VERSION_OLD}
重启策略
restart: always
环境变量
environment:
MYSQL_ROOT_PASSWORD: 123456 
端口映射
ports:
- 33061:3306 
文件挂载
volumes:
- my5Data:/var/lib/mysql 
- ./conf/my5.cnf:/etc/my.cnf   
网络链接 
networks:
- devNet  
```
### 2. mysql8
```
镜像
image: mysql:${MYSQL_VERSION}
重启策略
restart: always
环境变量
environment:
    MYSQL_ROOT_PASSWORD: 123456
端口映射   
ports:
    - 33060:3306  
文件挂载    
volumes:
    - my8Data:/var/lib/mysql  
    - ./conf/my8.cnf:/etc/my.cnf
网络链接    
networks:
    - devNet 
```
### 3. mongo
```
镜像
image: mongo:${MONGO_VERSION}
重启策略
restart: always
文件挂载
volumes:
    - mongoData:/data/db
    - ./conf/mongod.conf:/data/configdb/mongod.conf
端口映射
ports:
    - 27017:27017
启动容器执行的的命令，这里是通过挂载自定义配置文件启动mongod     
command: 
    mongod --config /data/configdb/mongod.conf
环境变量      
environment:
    MONGO_INITDB_ROOT_USERNAME: root
    MONGO_INITDB_ROOT_PASSWORD: 123456  
网络链接     
networks:
    - devNet 
```
### 4. mongo-express mongo界面管理工具
```
image: mongo-express:${MONGO_EXPRESS_VERSION}
restart: always
ports:
    - 8081:8081
environment:
    ME_CONFIG_MONGODB_ADMINUSERNAME: root
    ME_CONFIG_MONGODB_ADMINPASSWORD: 123456
    ME_CONFIG_MONGODB_URL: mongodb://root:123456@mongo:27017/  
networks:
    - devNet 
```
### 5. redis
```
image: redis:${REDIS_VERSION}   
restart: always 
ports:
    - 6379:6379
command:
    /bin/bash -c "redis-server /etc/redis.conf"  
networks:
    - devNet  
volumes:
    - redisData:/data   
    - ./conf/redis.conf:/etc/redis.conf  
rabbitmq:
image: rabbitmq:${RABBITMQ_VERSION}     
restart: always 
ports:
    - 15672:15672
    - 5672:5672
volumes:
    -  rabbitmqData:/var/lib/rabbitmq
environment:
    RABBITMQ_DEFAULT_USER: root  
    RABBITMQ_DEFAULT_PASS: 123456
```
### 6. postgres
```
image: postgres:${POSTGRES_VERSION}
restart: always
networks:
    - devNet
environment:
    POSTGRES_USER: root
    POSTGRES_PASSWORD: 123456
ports:
    - 5432:5432
command:
    postgres -c config_file=/var/lib/postgresql/data/postgresql.conf  
volumes:
    - postgresData:/var/lib/postgresql/data   
    - ./conf/pg_hba.conf:/var/lib/postgresql/data/pg_hba.conf
    - ./conf/postgresql.conf:/var/lib/postgresql/data/postgresql.conf
```
第一次启动需要先注释,因为数据库需要初始化，挂载进去文件存在会报错.
```
- ./conf/pg_hba.conf:/var/lib/postgresql/data/pg_hba.conf
    - ./conf/postgresql.conf:/var/lib/postgresql/data/postgresql.conf
```

### 7. mssql
```
image: mcr.microsoft.com/mssql/server:${MSSQL_VERSION}   
restart: always
networks:
    - devNet
environment:
    ACCEPT_EULA: Y
    MSSQL_SA_PASSWORD: Glkj2022!@#
ports:
    - 1401:1433
volumes:
    - mssqlData:/var/opt/mssql
```
### 8. php-fpm5
自己编译镜像
或者下载  docker pull wms3001/phpfpm5
```
使用官方镜像
image: php:${PHPFPM5_VERSION} 
应为扩展不全，自己打镜像后使用的自己镜像 
image: phpfpm5
如果没有打镜像，可以直接从dockerfile build
当前目录中存在Dockerfile文件
build：.
Dockerfile在其他目录中
build：
    context: ./dir
Dockerfile在其他目录中,并且名字不叫Dockerfile
build：
    context: ./dir    
    dockerfile: file
从git仓库拉去dockerfile，存在文件名是Dockerfile
build：
    context: https://github.com/wms3001/dockerFile.git
从git仓库拉去dockerfile，不存在文件名是Dockerfile 
build：
context: https://github.com/wms3001/dockerFile.git 
dockerfile: file

restart: always
networks: 
    - devNet
ports:
    - 9005:9000 
```
### 9. phpcli5
自己编译镜像
或者下载  docker pull wms3001/phpcli5
```
image: phpcli5
    restart: always
    networks: 
      - devNet
    stdin_open: true # -i interactive
    tty: true # -t tty
    privileged: true
    entrypoint: ["sh"] # 执行 sh
    volumes:
      - ./app/php5:/data   
      # - ./cron/phpcli5:/etc/crontab  
```
### 10. phpfpm7
自己编译镜像
或者下载  docker pull wms3001/phpfpm7
```
image: phpfpm7  
restart: always
networks: 
    - devNet
ports:
    - 9007:9000
```
### 11. phpcli7
自己编译镜像
或者下载  docker pull wms3001/phpcli7
```
image: phpcli7
restart: always
networks: 
    - devNet
stdin_open: true # -i interactive
tty: true # -t tty
privileged: true
entrypoint: ["sh"] # 执行 sh
volumes:
    - ./app/php7:/data   
    # - ./cron/phpcli7:/etc/crontab 
```
### 12. phpfpm8
自己编译镜像
或者下载  docker pull wms3001/phpfpm8
```
image: phpfpm8  
restart: always
networks: 
    - devNet
ports:
    - 9008:9000 
```
### 13. phpcli8
自己编译镜像
或者下载  docker pull wms3001/phpcli8
```
image: phpcli8
restart: always
networks: 
    - devNet
stdin_open: true # -i interactive
tty: true # -t tty
privileged: true
entrypoint: ["sh"] # 执行 sh
volumes:
    - ./app/php8:/data   
    # - ./cron/phpcli7:/etc/crontab   
```
### 14. nginx
```
image: nginx:${NGINX_VERSION}  
restart: always
networks: 
    - devNet
ports:
    - 8080:80
volumes:
    - ./app:/usr/share/nginx/html    
    - ./conf/nginx.conf:/etc/nginx/ngin
```
### 15. java
自己编译镜像
或者下载  docker pull wms3001/java19
```
image: oracle/jdk:19
command: "java -jar /app/package.jar"
restart: always
networks: 
    - devNet
ports:
    - 18889:18888  
volumes:
      - ./app/java:/app  
```