# 简单的部署java项目到linux服务器

## 打包

利用maven(gradle)插件完成,这里以maven为例子

- `mvn clean package` 清理输出目录并打包maven项目到输出目录(`springboo`中为`/target`目录)
- 打包完成后会得到一份`xxx.jar`文件

## 部署

### 解决环境依赖

- 数据库，缓存，消息队列等中间件服务环境

### 拷贝jar文件

- 利用`xftp`将jar文件拷贝到希望拷贝到的linux目录中

### 运行java项目

- 命令`nohup java -jar xxx.jar > /dev/null &` 后台运行java项目并且将控制台信息重定向到null（就是不重定向）

### 检查项目是否运行成功

- `netstat -tunpl | grep [端口号]` 查看是否有进行监听端口

### 关闭进程

- `kill -9 [pid进程号]`

### 监控

- 目前使用的是`spring-boot-admin`组件