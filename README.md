# MyUrls

本项目简单修改自大佬的项目，License应归属原项目作者： MIT © 2024 CareyWang
[原项目](https://github.com/CareyWang/MyUrls)
- 修改内容：
1. 修改了docker-compose.yaml文件，可以直接修改env文件使用现有redis
2. 新增了docker-compose_origin.yaml，如果没有自己部署redis，依然可以通过此yaml文件实现依赖于redisdocker的自部署
3. 增加了对redis数据库(默认DB：0)的选择，使用docker-compose.yaml方式部署可以设置
4. 增加了对AMH面板使用时的操作方法
5. .env文件的修改
> 基于 Go 1.22 与 Redis 实现的本地短链接服务，用于缩短 URL 与短链接还原。

# 目录
- Docker方式部署
- Install直接安装

# Docker方式部署
## Dependencies
本服务依赖于 Redis 提供长短链接映射关系存储，你需要本地安装 Redis 服务来保证短链接服务的正常运行。
现提供以下两个方式实现redis的部署：
AMH面板（宝塔类似）
命令行安装

### AMH面板操作
1. AMH面板上“缓存应用”中安装Reids和phpRedisAdmin
2. 点击面板的Reids配置，软件版本选择你的redis，设置监听地址不限制，视情况而定修改监听端口，【推荐】设置一个密码
3. 【参考】通过lngx创建站点，AMHSSL申请域名，申请后设置站点conf文件跨域：
   ```
    cd /home/wwwroot/Lngx环境/vhost/
    找到你的配置文件，https也需要
    添加 add_header 'Access-Control-Allow-Origin' '*';
    位置：location / {} 内的最后
    ```

### 命令行安装版
执行以下命令，详细端口、密码修改自行问AI或者google
```shell script
sudo apt-get update

# 安装Redis
sudo add-apt-repository ppa:chris-lea/redis-server -y 
sudo apt-get update 
sudo apt-get install redis-server -y 
```

## Docker 操作

### 使用docker-compose
[安装docker-compose](https://docs.docker.com/compose/install/)
```shell script
git clone https://github.com/TaoRone/MyUrls.git MyUrls
cd MyUrls
cp .env.example .env
# 修改.env文件内容，文件内容见下方
docker-compose up -d
# 或者docker compose up -d
```
.env文件修改内容：
```
MYURLS_PORT=9002 # docker映射端口号
MYURLS_DOMAIN=example.com # 你的域名，这里是用来加在短链接前面的
MYURLS_PROTO=https
MYURLS_REDIS_CONN=redis:6379 # 你的Redis数据库链接，必须带端口号
MYURLS_REDIS_PASSWORD=a12345 # 你的Redis数据库密码
MYURLS_REDIS_DB=0 # 你的Redis数据库DB号，默认为DB 0，详情可以从amh的管理面板进去查看
```

### 使用docker-compose_origin.yaml
[安装docker-compose](https://docs.docker.com/compose/install/)
```shell script
git clone https://github.com/TaoRone/MyUrls.git MyUrls
cd MyUrls
cp .env.example .env
# 修改.env文件内容，文件内容见下方
docker-compose -f docker-compose_origin.yaml up -d
# 或者docker compose -f docker-compose_origin.yaml up -d
```
.env文件修改内容：
```
MYURLS_PORT=9002 # docker映射端口号
MYURLS_DOMAIN=example.com # 你的域名，这里是用来加在短链接前面的
MYURLS_PROTO=https
MYURLS_REDIS_CONN=redis:6379 # 你的Redis数据库链接，必须带端口号
MYURLS_REDIS_PASSWORD=a12345 # 你的Redis数据库密码
```
### 如果想要重新部署
```
docker-compose down
docker-compose up -d
# 或者 docker-compose -f docker-compose_origin.yaml up -d
```

### 如果想要清理编译过程中产生不需要的文件
```
docker system prune -a --volumes
```


### 使用docker CLI（不支持修改redis数据库DB号）
这个镜像这里列举的是原版提供的，因此不能修改reids的DB号
```
docker run -d --restart always --name myurls careywong/myurls:latest -domain example.com -port 8002 -conn 127.0.0.1:6379 -password ''
```

# Install直接安装
以下内容为原项目方式，未修改
- 安装项目依赖
```shell script
make install
```
- 生成可执行文件，目录位于 build/ 。默认当前平台，其他平台请参照 Makefile 或执行对应 go build 命令。
```shell script
make
```
## Usage使用
- 前往 [Actions](https://github.com/CareyWang/MyUrls/actions/workflows/go.yml) 下载对应平台可执行文件。
```shell script
Usage of ./MyUrls:
  -conn string
        address of the redis server (default "localhost:6379")
  -domain string
        domain of the server (default "localhost:8080")
  -h    display help
  -password string
        password of the redis server
  -port string
        port to run the server on (default "8080")
  -proto string
        protocol of the server (default "https")
```

- 建议配合 [pm2](https://pm2.keymetrics.io/) 开启守护进程。
```shell script
pm2 start myurls --name myurls -- -domain example.com
```

## 日志清理
假定工作目录为 `/app`，可基于 logrotate 配置应用日志的自动轮转与清理。可参考示例配置，每天轮转一次日志文件，保留最近7天
```shell 
tee > /etc/logrotate.d/myurls <<EOF
/app/logs/access.log {
    daily
    rotate 7
    missingok
    notifempty
    compress
    delaycompress
    copytruncate
    create 640 root adm
}
EOF

# 测试是否正常工作，不会实际执行切割
logrotate -d /etc/logrotate.d/myurls
```


