### 定时任务

> CODO项目定时任务模块，定时任务完全兼容crontab，支持到秒级

**下载代码**
```shell
echo -e "\033[32m [INFO]: codo_cron(定时任务) Start install. \033[0m"
if ! which wget &>/dev/null; then yum install -y wget >/dev/null 2>&1;fi
if ! which git &>/dev/null; then yum install -y git >/dev/null 2>&1;fi
[ ! -d /opt/codo/ ] && mkdir -p /opt/codo
cd /opt/codo && git clone https://github.com/opendevops-cn/codo-cron.git
cd codo-cron
```



**修改配置**
> 同样，这里codo-task也支持取env环境变量，建议还是修改下默认配置
```shell
#导入环境变量文件，最开始准备的环境变量文件
source env.sh
#后端数据库名称,建议不要修改，初始化data.sql已经指定了数据库名字，若需改请一块修改
CRON_DB_DBNAME='codo_cron' 

#定时任务域名，这里不要域名，只启动一个端口，用本机IP
#sed -i  "s#server_name .*#server_name ${task_domain};#g" doc/nginx_ops.conf 

sed -i "s#cookie_secret = .*#cookie_secret = '${cookie_secret}'#g" settings.py
sed -i "s#token_secret = .*#token_secret = '${token_secret}'#g" settings.py

#mysql配置
sed -i "s#DEFAULT_DB_DBHOST = .*#DEFAULT_DB_DBHOST = os.getenv('DEFAULT_DB_DBHOST', '${DEFAULT_DB_DBHOST}')#g" settings.py
sed -i "s#DEFAULT_DB_DBPORT = .*#DEFAULT_DB_DBPORT = os.getenv('DEFAULT_DB_DBPORT', '${DEFAULT_DB_DBPORT}')#g" settings.py
sed -i "s#DEFAULT_DB_DBUSER = .*#DEFAULT_DB_DBUSER = os.getenv('DEFAULT_DB_DBUSER', '${DEFAULT_DB_DBUSER}')#g" settings.py
sed -i "s#DEFAULT_DB_DBPORT = .*#DEFAULT_DB_DBPORT = os.getenv('DEFAULT_DB_DBPORT', '${DEFAULT_DB_DBPORT}')#g" settings.py
sed -i "s#DEFAULT_DB_DBPWD = .*#DEFAULT_DB_DBPWD = os.getenv('DEFAULT_DB_DBPWD', '${DEFAULT_DB_DBPWD}')#g" settings.py
sed -i "s#DEFAULT_DB_DBNAME = .*#DEFAULT_DB_DBNAME = os.getenv('DEFAULT_DB_DBNAME', '${CRON_DB_DBNAME}')#g" settings.py

#只读MySQL配置
sed -i "s#READONLY_DB_DBHOST = .*#READONLY_DB_DBHOST = os.getenv('READONLY_DB_DBHOST', '${READONLY_DB_DBHOST}')#g" settings.py
sed -i "s#READONLY_DB_DBPORT = .*#READONLY_DB_DBPORT = os.getenv('READONLY_DB_DBPORT', '${READONLY_DB_DBPORT}')#g" settings.py
sed -i "s#READONLY_DB_DBUSER = .*#READONLY_DB_DBUSER = os.getenv('READONLY_DB_DBUSER', '${READONLY_DB_DBUSER}')#g" settings.py
sed -i "s#READONLY_DB_DBPWD = .*#READONLY_DB_DBPWD = os.getenv('READONLY_DB_DBPWD', '${READONLY_DB_DBPWD}')#g" settings.py
sed -i "s#READONLY_DB_DBNAME = .*#READONLY_DB_DBNAME = os.getenv('READONLY_DB_DBNAME', '${CRON_DB_DBNAME}')#g" settings.py

#redis配置
sed -i "s#DEFAULT_REDIS_HOST = .*#DEFAULT_REDIS_HOST = os.getenv('DEFAULT_REDIS_HOST', '${DEFAULT_REDIS_HOST}')#g" settings.py
sed -i "s#DEFAULT_REDIS_PORT = .*#DEFAULT_REDIS_PORT = os.getenv('DEFAULT_REDIS_PORT', '${DEFAULT_REDIS_PORT}')#g" settings.py
sed -i "s#DEFAULT_REDIS_PASSWORD = .*#DEFAULT_REDIS_PASSWORD = os.getenv('DEFAULT_REDIS_PASSWORD', '${DEFAULT_REDIS_PASSWORD}')#g" settings.py

#删除没必要的目录映射
#这是测试用到的，先删除掉
name=/var/www
sed -i 's#'$name'#EXCLUSIVE#;/EXCLUSIVE/d' docker-compose.yml
name_test=/var/www/do_cron/db_sync.py
sed -i 's#'$name_test'#EXCLUSIVE02#;/EXCLUSIVE02/d' Dockerfile

```

**初始化数据**

```shell
#初始化SQL
wget https://raw.githubusercontent.com/opendevops-cn/opendevops/master/sql/codo_cron-beta0.2.sql
#创建数据库
mysql -h127.0.0.1 -uroot -p${MYSQL_PASSWORD}
create database `codo_cron` default character set utf8mb4 collate utf8mb4_unicode_ci;
#导入数据
mysql -h127.0.0.1 -uroot -p${MYSQL_PASSWORD} codo_cron < codo_cron-beta0.2.sql
#确认
mysql -h127.0.0.1 -uroot -p${MYSQL_PASSWORD} codo_cron -e  "show tables;"

```





**编译，启动**

```shell
#编译镜像
docker build . -t do_cron_image

#启动
docker-compose up -d
```
**测试**

> 日志文件位置统一：/var/log/supervisor/

```
01. 查看日志
tailf /var/log/supervisor/cron.log   #确认没报错
```


