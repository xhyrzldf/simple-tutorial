# 如何部署spring-boot-admin监控组件

## 监控的本质

监控服务分为监控服务器(服务端)与被监控服务器(客户端)，服务端通过轮询客户端服务器可以提供一些健康信息的接口(这些接口由`spring-boot-starter-actuator`组件提供)，将轮询到的信息以图形化的方式展现出来，形成监控。

## 服务端

- 服务端的教程随后更新,目前已经部署在内网机器中
  - 测试环境地址 : `192.168.1.3:8000/ds-monitor`

## 客户端

### 添加客户端依赖

```xml
        <!-- 健康信息接口提供组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!--spring-boot-admin-客户端 依赖-->
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-client</artifactId>
            <version>1.5.7</version>
        </dependency>
```

### 客户端相关配置

#### actuator 健康信息相关配置

为了提供健康信息的接口与正常的服务接口错开，这里可以指定额外的端口与服务路径前缀，同时为了安全性，可以打开安全服务，需要验证相关信息后才可访问(教程测试环境并没有打开)。

```properties
## 开启actuator
management.endpoints.enabled-by-default=true
management.security.enabled=false
## 监控api端口号
management.server.port=8083
## 监控的api前缀
management.context-path=/actuator
```

#### 配置监控端的服务端地址

既然是客户端，即被监控的一方，因此理所当然的要配置监控端服务器的地址(服务端教程里提供的地址`192.168.1.3:8000`)，这里强烈建议为`dev`与`prod`环境配置不同的注册机制，这样可以防止无用的本地测试客户端注册到服务端，造成无必要的访问。

```properties
# spring-boot-admin 监控配置
spring.application.name=debug-server(centOS-01)
spring.boot.admin.url=http://192.168.1.3:8000/ds-monitor
```

以上的配置分别是设置该客户端向服务端注册提供的客户端名称(客户端名称请改成自己的名称)与服务端地址

## 最后

如此就注册监控服务成功了，教程中提供的配置是为最简配置，详细的配置与教程请查看[官方教程](https://github.com/codecentric/spring-boot-admin)