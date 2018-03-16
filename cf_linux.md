# 基于 Linux 的安装

[TOC]



最小运行环境:

- Java 8 (JRE or JDK)
- 至少 1 G 的内存
- 1 G 以上的空间

推荐运行环境: 

- Java 8 (JRE or JDK)
- 4 G 以上的内存
- 50 G 以上的空间




## 1. 通过源码安装 flow.ci platflom 后端程序

### 1.1 安装前环境准备

**Java**

flowci 是基于 Java 1.8 版本的应用程序，所以需要在机器上安装 Java 1.8 或以上的版本。如果不确定现有的 Java 版本，可以通过 `java -version` 命令检查当前的 Java 环境。

```shell
# Ubuntu 安装方式
## 使用 apt-get 安装，然后配置 JAVA_HOME/PATH
sudo apt-get install openjdk-8
export JAVA_HOME=${JDK_INSTALL_HOME}
export PATH=$JAVA_HOME/bin:$PATH

## 官网下载 tar.gz 的包，解压到指定目录，然后配置 JAVA_HOME/PATH
export JAVA_HOME=${JDK_INSTALL_HOME}
export PATH=$JAVA_HOME/bin:$PATH
```

**Git**

flowci 目前只支持基于 Git 的版本控制系统，所以需要在机器上安装 Git，可以通过 `git --version` 命令检查 Git 是否安装。

```Shell
# Ubuntu 安装方式
apt-get install git
# git config...
```

**Tomcat**

Tomcat 8.5 以上，下载 tar.gz 的包，并解压。

**Maven**

使用 maven 3.3.5 版本

```shell
sudo apt-get install maven   # maven_home: /usr/share/maven
```

**Nginx**

```shell
sudo apt-get install nginx.  # nginx_conf: /etc/nginx/nginx.conf
```

**MySQL**

MySQL 5.6 以上

> 提示：确保 MySQL 的时区与服务器保持一致

```shell
# Ubuntu 安装方式
sudo apt-get install mysql-server
sudo mysql_secure_installation

# 基本命令
systemctl status mysql.service
systemctl start mysql.service
mysql -u root -p
```



### 1.2 获取源码并编译

> 提示: flowci 使用 maven 编译，3.0 以上版本， 可以在命令行中输入 `mvn -v` 检查 maven 是否正确安装。

  通过以下命令，编译所需要的 war / jar 包，结果输出在 ./dist 文件夹下

  ```shell
git clone -b master https://github.com/FlowCI/flow-platform.git
cd flow-platform
mvn clean package -DskipTests=true
ls ./dist

## Output
flow-api-v0.1.4-alpha.war             # flow-api
flow-control-center-v0.1.4-alpha.war  # flow-cc
flow-agent-v0.1.4-alpha.jar           # flow-agent
  ```



### 1.3 修改配置文件

1. 创建数据库

  flow.ci 后端需要两个数据库对应不同的服务，可以执行以下语句在 MySQL 中创建所需要的数据库。

  ```sql
  mysql -u root -p -e "CREATE DATABASE flow_api_db CHARACTER SET utf8 COLLATE utf8_bin;"
  mysql -u root -p -e "CREATE DATABASE flow_cc_db CHARACTER SET utf8 COLLATE utf8_bin;"
  ```

2. 导入数据库表结构

  ```sql
  mysql -u root -p --database=flow_api_db < flow-platform/schema/flow_api_db.sql
  mysql -u root -p --database=flow_cc_db < flow-platform/schema/flow_cc_db.sql
  ```

3. 更改及设置配置文件

  **config/app-api.properties**

  - `jdbc.url`:                  配置 MySQL 数据库地址 (127.0.0.1:3306)
  - `jdbc.username`:        配置 MySQL 的用户名
  - `jdbc.password`:        配置 MySQL 的密码
  - `jdbc.pool.size`:      配置连接池数量，推荐 50 以上
  - `api.workspace`:        配置数据存储目录
  - `api.run.indocker`:  是否通过 docker 启动
  - `domain.api`:              配置 API 服务的 URL 地址 (http://127.0.0.1:8080/flow-api)
  - `domain.web`:              配置 Web 端的 URL 地址 (http://127.0.0.1:443)
  - `domain.cc`:                配置 CC 服务的 URL 地址 (http://127.0.0.1:8080/flow-control-center)
  - `task.job.toggle.execution_timeout`:                                   是否开启 Job 过期检查
  - `task.job.toggle.execution_create_session_duration`:   设置创建 Job 时的过期时间(秒)
  - `task.job.toggle.execution_running_duration`:                 设置 Job 运行时的过期时间(秒)

  **config/app-cc.properties**

  - `jdbc.url`:                       配置 MySQL 数据库地址 (127.0.0.1:3306)

  - `jdbc.username`:             配置 MySQL 的用户名

  - `jdbc.password`:             配置 MySQL 的密码

  - `jdbc.pool.size`:           配置连接池数量，推荐 50 以上

  - `zk.server.embedded`:   是否使用 flow.ci 内嵌的 Zookeeper 服务，true or false

  - `zk.host`:                         Zookeeper 的地址，需要外网访问权限

  - `agent.config.ws`:         配置 Websocket 的地址，必须以 ws 开头，和 API 中 `domain.api` 的域名一致

  - `agent.config.cc`:         配置 CC 的 地址，需要和 API 中 `domain.cc` 地址一致

    ​

  > 根据用户所设置信息，需要修改 API 配置文件中的 MySQL 设置， 以及对应的 domain 地址的设置。更详细的配置文件说明，请参见 <a>配置文件说明</a>

### 1.4 导入配置文件并启动

1. 安装 dist 目录下的 *flow-api.war* 以及 *flow-control-center.war* 到 Tomcat 服务器 webapps 中
2. 设置如下两个环境变量 **(存在不生效的情况，所以建议修改 webapps 中的 properties 文件)**
   - FLOW_API_CONFIG_PATH=${LOCATION}/app-api.properties
   - FLOW_CC_CONFIG_PATH=${LOCATION}/app-cc.properties
3. 启动 tomcat，验证是否成功
   - http://127.0.0.1:8080/flow-api/index    验证 api 接口是否返回
   - http://127.0.0.1:8080/flow-cc/index      验证 cc   接口是否返回




## 2. 源码安装 flow.ci web 页面

flowci 采用的为前后端分离的结构，所以需要用户在安装后台服务后，安装 web 界面。

### 2.1 安装前的准备

**node.js**

Node.js v6.6.0 以上版本

```shell
sudo apt-get install nodejs
```

**npm**

npm v3 以上版本

```shell
sudo apt-get install npm/cnpm
```


### 2.2 获取源码并编译

通过以下脚本编译 web 项目，编译成功后，请在 dist 文件夹下查看编译好的项目文件。

> 请先根据配置的 API URL 替换 `FLOW_WEB_API` 环境变量。

```shell
git clone -b master https://github.com/FlowCI/flow-web.git

# 编译前安装依赖包
cd flow-web
npm install
# 额外需要手动安装的包:
cnpm install \
	fs-extra \
	chalk \
	figures \
	webpack \ 
	html-webpack-plugin \
	extract-text-webpack-plugin \
	babel-core \
	babel-loader \
	eslint \
	eslint-loader
	eslint-plugin-react \
	eslint-plugin-babel \
	eslint-config-standard-react
cnpm install

export NODE_ENV=production
export FLOW_WEB_API="http://127.0.0.1:8080/flow-apis"

npm run compile

# Outputs:
dist/index.html
dist/*.js
```


### 2.3 Nginx 部署

web 可以部署在任意 http 应用服务器上，如 Nginx, Apache Http Server 等。
> 提示: web 项目使用 react 构建，需要在部署到 http 应用服务器后，配置服务器的路由，否则会无法刷新页面。如在 Nginx 上，需添加配置 `try_files $uri /index.html;`

```nginx
# nginx 配置 /usr/local/etc/nginx/nginx.conf
http {
    server {
        listen       443;
        location / {
            root   /opt/flowci/flow-web/dist/;
            try_files $uri /index.html;
        }
    }
}
```



**nginx 使用**

```shell
# 启动
nginx 

# reload
nginx -s reload
```





## 3. 创建并启动 Agent

**只有在创建了 Agent 并启动成功之后，工作流才能够正常的运行，否则会一直处于准备中的状态**

**Agent 使用方式见 [Agent管理](./admin_agent.md)**

