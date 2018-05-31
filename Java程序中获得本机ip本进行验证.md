# Java程序中获得本机ip本进行验证

## 思路

使用`java.net`包下的接口获得本地网络的一些信息(网卡,ip,mac地址等等)，再使用正则表达式对字符串进行过滤获得自己想要的一些ip

## 实现

### 获得ip

```java
    public static void main(String args[]) {
        try {
            Enumeration allNetInterfaces = NetworkInterface.getNetworkInterfaces();
            InetAddress ip;
            while (allNetInterfaces.hasMoreElements()) {
                NetworkInterface netInterface = (NetworkInterface) allNetInterfaces.nextElement();
                Enumeration addresses = netInterface.getInetAddresses();
                while (addresses.hasMoreElements()) {
                    ip = (InetAddress) addresses.nextElement();
                    if (ip instanceof Inet4Address) {
                        System.out.println("本机的IP=" + ip.getHostAddress());
                    }
                }
            }
        } catch (Exception ex) {
        }
    }
```

核心代码如图，首先获得网卡，再根据网卡遍历出ip对象,再筛选出对应的ip类型(ipv4 or ipv6)，要注意好对异常的处理。

### 过滤

查询出ip包含多个网卡提供的ip（本机,局域网,外网等等），使用正则表达式过滤出自己需要的ip

```java
RegexUtil.check("192\\.168\\.1\\.[0-9]{0,3}", ip.getHostAddress())
```

工具类在框架下的`utils`包下,或者使用原生的

```java
Pattern.matches(regex, s)//regex=正则表达式,s=待检测的字符串
```

## 例子

这是一段简单的例子,用于获取ip并过滤出局域网ip

```java
    public String getAreaNetWorkIp() {
        try {
            Enumeration allNetInterfaces = NetworkInterface.getNetworkInterfaces();
            InetAddress ip;
            while (allNetInterfaces.hasMoreElements()) {
                NetworkInterface netInterface = (NetworkInterface) allNetInterfaces.nextElement();
                Enumeration addresses = netInterface.getInetAddresses();
                while (addresses.hasMoreElements()) {
                    ip = (InetAddress) addresses.nextElement();
                    if (ip instanceof Inet4Address && RegexUtil.check("192\\.168\\.1\\.[0-9]{0,3}", ip.getHostAddress())) {
                        System.out.println("本机的局域网IP=" + ip.getHostAddress());
                        return ip.getHostAddress();
                    }
                }
            }
        } catch (SocketException e) {
            System.out.println("获取局域网ip失败,默认返回空字符串");
        }
        return "";
    }
```