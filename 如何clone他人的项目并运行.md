#  如何顺利的clone别人的项目并且运行

## 明确目的
- 单纯的clone运行
- 单项目的合作开发
- 多人开源项目开发 fork and pull request

## how to do

### 下载
- git clone [地址]

### 解决依赖
#### 外部依赖
数据库,redis,消息队列,ES等中间件组件
#### maven(构建)依赖
**这里以maven做例子**
  1.设置maven
  2.然后重新导入maven Reimport
  3.观察pom文件查看是否通过检查
   - 检查pom文件有没有语法错误
   - 检查自己仓库的jar是否下载完整了,如果不完整就重新下载
   - 检查jar包是否是maven中央仓库所有的,通过groupId与artifactId去寻找,有没有这个版本，还说这个版本是在一些私人库

   4.重新导入(`Reimport`)
   5.运行项目
