# Uubuntu 18.04 LTS 安装mongoDB

[TOC]



最近在查阅mongodb的相关资料，搭建实验环境时，发现官网上的ubuntu安装教程只写了支持14.04与16.04 lts的版本，于是查阅了相关资料，这里整理一下unbuntu 18.04安装mongodb的方法。

## 注意点

- 官网已经声明，ubuntu默认仓库的mongodb的库不是官方维护的，推荐使用官方维护库的mongodb进行使用。

## 安装步骤

### 添加包秘钥

```shell
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2930ADAE8CAF5059EE73BB4B58712A2291FA4AD5
```

### 添加mongoDB官方仓库

```shell
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.6 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.6.list
```

### 更新源并安装mongoDB

```shell
sudo apt update
sudo apt install -y mongodb-org
```

## 启动mongoDB服务

使用如下命令启动/关闭/开机自启 mongoDB

```shell
sudo systemctl stop mongod.service
sudo systemctl start mongod.service
sudo systemctl enable mongod.service
```

### 检查运行状态

启动之后,使用如下命令检查mongodb运行状态

```shell
sudo systemctl status mongod
```

状态正常的话会出现类似如下的信息

```powershell
● mongod.service - High-performance, schema-free document-oriented database
   Loaded: loaded (/lib/systemd/system/mongod.service; disabled; vendor preset: 
   Active: active (running) since Mon 2018-08-06 15:48:24 CST; 1min 54s ago
     Docs: https://docs.mongodb.org/manual
 Main PID: 3215 (mongod)
   CGroup: /system.slice/mongod.service
           └─3215 /usr/bin/mongod --config /etc/mongod.conf

```

### 连接mongoDB shell窗口

mongoDB 默认运行在27017端口下,使用如下命令进行shell连接

```shell
mongo --host 127.0.0.1:27017
```

如果正常的话，应当会出现如下的信息

```shell
MongoDB shell version v3.6.6
connecting to: mongodb://127.0.0.1:27017/
MongoDB server version: 3.6.6
Server has startup warnings: 
2018-08-06T15:48:25.036+0800 I STORAGE  [initandlisten] 
```

### 创建账户

如果你想要创建自己的管理员权限账户，连接shell后，使用如下命令

```shell
> use admin
> db.createUser({user:"账户名,例如admin", pwd:"密码,例如admin", roles:[{role:"root", db:"admin"}]})
```

创建成功后会如下类似如下的信息

```shell
Successfully added user: {
	"user" : "admin",
	"roles" : [
		{
			"role" : "root",
			"db" : "admin"
		}
	]
}
```

至此，基本安装教程就完毕了。

## 参考资料

[Install MongoDB On Ubuntu 18.04 LTS Server](https://websiteforstudents.com/install-mongodb-on-ubuntu-18-04-lts-beta-server/)