## 阿里云DDNS



##### 通常要挂一台自己的服务器，可能有以下解决：

1.租用云服务器（成本太高） 

2.手动搭建一台

搭建一台服务器无非就是想通过外网可以随时访问到服务器，通常第一步我们得打通网络环境，如：

1.直接访问（固定公网ip）

2.代理访问（vpn、frp、某某123、某某壳、等） 

根据使用场景以及成本选择不同方案，代理访问速度往往没有直接访问流畅，尤其有些第三方平台有带宽限制 

本文介绍如何使用普通宽带与家用电脑搭建一台服务器，主要解决以下问题：

1.家用网络端口映射

2.固定ip地址



### 准备

##### 1.宽带改成公网ip

这个只是动态分配的ip,不同于固定ip，默认情况运营商可能分配的私有ip，找客服改一下就行了

##### 2.光猫模式选择

现在运营商的光猫都自带拨号跟路由功能，通常情况装机人员使用的是路由模式，装机人员会帮你配置宽带拨号信息在光猫里，连上lan口就能上网，如果说给的光猫能进行有效的端口映射也无妨，不能则需要改成桥接模式，然后自己在光猫下级用可端口映射的路由器设置拨号，关键点就是要端口能映射出去

##### 3.租用阿里云域名

由于普通宽带分配的公网ip是动态的，重启路由或者过一段时间就会重新分配一个ip，所以要保证域名解析的ip是当前分配的ip；在阿里云注册个域名，便宜的1年几块钱不等

##### 4.动态更改阿里云域名解析

本地外网ip同步到阿里云域名解析，达到动态解析ip地址目的 

阿里云有相关sdk管理域名解析，分2种：

一种以http规范的API文档

一种以java语言封装的jar包



### 实现

#### 思路

##### 1.获取公网ip地址

##### 2.根据解析记录类型查询阿里云解析中所有解析记录

##### 3.遍历解析记录并根据主机记录添加或更新解析记录

##### 4.等待生效（通常修改完不会立马生效）

##### 5.光猫或者路由器上配置端口映射

##### 6.域名加端口进行访问（80或8080端口可能被限制了，建议用10000以后的端口）

主要代码

<https://github.com/GhostMouse7369/ddns/blob/master/ddns-starter/src/main/kotlin/top/laoshuzi/ddns/runner/DdnsRunner.kt> 

```kotlin
//创建协程
val launch = GlobalScope.launch {
        delay(1000)
        while (true) {
            println("==============[${getNowTime()}]==============")

            // 获取Ip
            var ip: String? = null

            println("来自[${ddnsProperties.config.ipUrl}]")
            for (i in 1..3) {
                if (!StringUtils.isEmpty(ip))
                    break
                println("获取IP地址:第${i}次")
                ip = ipService.getLocalOutIp()
            }

            if (StringUtils.isEmpty(ip))
                throw Exception("无法获取IP地址")

            // 遍历配置
            ddnsProperties.domain.forEach { domainProperties ->
                val domain = domainProperties.value
                // 获取A记录
                val records = domainRecordsService.getDomainRecordsByType(domain.domainName, "A")
                        .associateBy {
                    // 转化成Map
                    it.rR
                }
                // 遍历解析记录
                domain.rrs.forEach { rr ->
                    println("同步解析记录-->$rr.${domain.domainName}")
                    val record = records[rr]
                    if (record == null) {
                        // 添加记录
                        val addRecordDTO = AddRecordDTO().apply {
                            domainName = domain.domainName
                            type = "A"
                            rR = rr
                            value = ip
                        }
                        println("添加解析记录-->${addRecordDTO.toString()}")
                        domainRecordsService.addDomainRecord(addRecordDTO)
                    } else {
                        // 更新记录
                        if (record.value == ip) {
                            println("IP地址无需变更")
                        } else {
                            val updateRecordDTO = UpdateRecordDTO().apply {
                                recordId = record.recordId
                                type = record.type
                                rR = record.rR
                                value = ip
                            }
                            println("更新解析记录-->${updateRecordDTO.toString()}")
                            domainRecordsService.updateDomainRecord(updateRecordDTO)
                        }
                    }
                }

            }

            println("=================================================")
            delay(300000)
        }
}
```

###### application.yml 

```yaml
top:
  laoshuzi:
    ddns:
      config:
        regionId: cn-hangzhou
        accessKeyId: <accessKeyId>
        accessKeySecret: <accessKeySecret>
        IpUrl: http://ip-api.com/json
      domain:
        laoshuzi:
          domainName: <domainName>
          rrs:
          - <rr>
```



**固定ip地址代码** 

###### jvm版 git：<https://github.com/GhostMouse7369/ddns> 



未完
