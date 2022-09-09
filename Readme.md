# Topology-Scanner

<img src="https://wedoc.canway.net/imgs/img/嘉为蓝鲸.jpg" >


Topology-Scanner是WeOps团队免费开放的一个网络拓扑自动扫描模块，可以自动发现网络设备的类型、网络设备之间的互联关系

#### 更多资料/工具包下载可见“蓝鲸 S-mart市场”
https://bk.tencent.com/s-mart/application/282/detail

#### 更多问题欢迎添加“小嘉”微信，加入官方沟通群

<img src="https://wedoc.canway.net/imgs/img/小嘉.jpg" width="30%" height="30%">


## 使用方式

```
java -jar ./topology-scanner.jar --config_path=./config/
```

## 拓扑发现算法
### 常规算法
常规算法即为 AFT 算法，是基于 AFT 建立链路。算法原理如下 ：
由于在发现设备的过程中，设备的ip和所在的子网都会被记录下来。在确定拓扑时， 根据发现过程中记录下的每个子网，首先确定交换域，交换域的定义为：交换域建模为一棵无向树G=(D,E). D是交换域内所有结点(网络设备)的集合.E是设备端口之间的直接连接集合, S表示所有交换机的集合,H表示所有主机的集合,D=S+H.(路由器算作主机)。最终对交换域进行合并，根据AFT 建立链路。参数运行配置中如果开启了使用ARP表，那还会再根据ARP 表来补充未发现的链路。
在发现过程中需要获取或扫描网络设备上的以下几个表： IPAdress 表，IFTable 表， ARP 表， FDB 表。

### CDP算法
适用于支持CDP协议的网络设备，如思科设备。是基于 CDP 协议来发现链路。CDP 算法原理如下 ：根据 CDP 协议从所有发现出来的设备中选出具有CDP 协议功能的设备，这是通过snmp 探测器去采集设备上的CDP 表，由此来判断是否有CDP协议能力。在利用CDP发现链路时，不需要只有路由、路由交换才具有的特征, 而且只能发现网络设备之间的连接。通过CDP表和IFTabel 表结合来确定设备的拓扑链路。由于CDP 的局限性，对拓扑发现结果精确度会影响。
在发现过程中需要获取或扫描网络设备上的以下几个表： CDP 表,  IFTable 表。

### 桥接算法
桥接算法是根据网络设备上的多个表相结合的方式来发现链路。桥接算法比其它的几个拓扑算法要复杂一些，参与计算的表要多。在发现的所有设备中，首先根据网络设备的 FDB 表来定义域，在划分好多个域后，确定域内的设备的连接，结合 FDB 表，IFTable 表、IPAdress 表 、basePortTable表 确定设备之间的转发与交换路径。通过STP 表生成一个无环有向图，即最终的拓扑链路。
在发现过程中需要获取或扫描网络设备上的以下几个表： basePortTable 表,  STP 表, FDB 表,IPAdress 表,  IFTable 表。

### LLDP 算法
LLDP 算法是根据 LLDP 表来确定网络设备的链路。算法原理如下 ： 从发现的所有设备中筛选出具有 LLDP 功能的设备。这步是通过 snmp 探测器获取 设备上的 LLDP 表来判断的。由于具有LLDP 协议的只能是交换机或路由器等网络设备，像防火墙，主机这种类型的设备如果没有开启LLDP协议，是不参与拓扑计算的。通过LLDP 表和 IFTable 表结合，来确定设备的拓扑。
在发现过程中需要获取或扫描网络设备上的以下几个表： LLDP 表 , IFTable 表。


## 配置说明

### 拓扑发现请求参数文件(application.yml)

#### ips

[全网发现] 模式时，为必填项。核心设备的ip, 多个ip 用逗号隔开。range 参数选填,起过滤作用。eg: 192.168.1.0,192.168.2.0

[子网发现] 模式时，为选填项。子网ip地址和掩码，必须成对。可多个，逗号隔开。 若为子网发现， ips 参数和range 参数不能同时为空。详见子网发现方式。 eg: 192.168.1.0,255.255.255.0,
192.168.2.0,255.255.255.0

#### hop

搜索深度，必填

#### group

使用SNMP V2协议时必填，SNMP V2的团体名，多个团体名用逗号隔开 eg: public,Huawei-public 当使用SNMP V3协议时可不填

#### range

[全网发现] 模式时，为选填项。Ip 范围,起过滤作用，可以多对，每对之间用; 号分隔，由开始和结束组成。eg: 192.168.1.0,192.168.1.255;192.168.2.0,192.168.2.255

[子网发现] 模式时，为选填项。若发现方式为子网发现，ips 参数和range 参数不能同时为空。Ip 范围,相当于范围发现，与子网发现结果取并集。Ip范围可以多对，每对由开始和结束组成，每对之间用;号分隔，eg:
192.168.1.0,192.168.1.255;192.168.2.0,192.168.2.255

#### way

发现方式： 0-全网发现 1-子网发现

#### algory

发现算法：

* 0-常规算法
* 1-CDP算法
* 2-LLDP 算法
* 3-桥接算法

#### version

SNMP 版本号，

* 2-SNMP 版本1或2
* 3-SNMP 版本3

#### v3

当使用SNMP V1/V2版本时可不填，当使用V3时，可填写如下JSON

```
[{
    username：用户名, 根据 safeLevel 级别选填,
    safeLevel：安全级别,必须为以下三者之一
              NOAUTH_NOPRIV  // 无认证无加密
              AUTH_NOPRIV    // 有认证无加密
              AUTH_PRIV      // 有认证有加密,
    protocol: 认证协议, 根据 safeLevel 级别选填, 必须为以下两者之一
              AuthMD5     // MD5
              AuthSHA     // SHA,
    pwd:     认证密码，根据 safeLevel 级别选填 ,
    encrypt:      加密协议，根据 safeLevel 级别选填，必须为以下5者之一
                PrivDES       // DES 加密
                Priv3DES      // 3DES 加密
                PrivAES128    // AES 128位
                PrivAES192    // AES 192位
                PrivAES256    // AES 256位,
    encryptPwd： 加密密码,根据 safeLevel 级别选填,
    content: 上下文,
    port: SNMP 端口号
}]
```

示例参数如下

```
[
          {
               "userName":"",                        //用户名
               "safeLevel":"NOAUTH_NOPRIV",          //安全级别
               "protocol":"",                        //认证协议
               "pwd":"",                             //认证密码
               "encrypt":"",                         //加密协议
               "encryptPwd":"",                     //加密密码
               "context":"",                        //上下文
               "port":"161"                        //端口
           }
  ]

```

### 拓扑发现运行的参数文件(discovery.properties)

参数    |类型    |说明|
----|------|---|
discovery.pool.max    |int|    拓扑发现线程池最大线程数，默认：256
discovery.pool.min|    int|    拓扑发现线程池最小线程数，默认：50
discovery.pool.queue.size|    int|    拓扑发现线程池队列，默认：500
discovery.min.maskbit    |int    |扫描的最小子网掩码位数,对子网扫描不起作用,默认：16
discovery.ping.batch    |int|    Ping 批次大小
discovery.ping.enable    |bool|    使用snmp探测前是否使用ping 来初次过滤，默认：false
discovery.ping.delay    |int|    Ping 重试延迟时间，单位毫秒，默认：1000
discovery.ping.retry    |int|    Ping 重试次数。默认：1
discovery.device.timeout    |int|    单个设备发现超时时间，单位分钟，默认：3
link.distinct    |bool    |两条设备之间如果存在多条链路，是否去重。默认：false
link.remove.cycle    |bool    |设备之间链路，是否过滤掉环路，默认：true
link.use.arp    |bool|    设备链路时，是否使用ARP 表建立链路。默认：false
link.cross.subnet    |bool|    设备链路时，是否跨子网。默认：true
discovery.snmp.retry|    int    |Snmp 探测时，单次报文重试次数。默认：1
discovery.snmp.timeout    |int|    Snmp 探测时，单次报文的超时时间。单位毫秒，默认：1500
discovery.snmp.detect.retry.number|    int    |单个ip 在snmp 探测失败时，重试次数，默认：2

### 设备oid 与设备类型字典文件(systemoid.xml/getterConfig.xml)

为了能更精确的采集网络设备上的各种表，特别是 FDB 表， 由于设备类型不同，FDB 表采集所用的 oid 也有差别。 通过外部 getterConfig.xml 文件来指定某种设备采集的SNMP 采集器。默认
getterConfig.xml 配置的getters 子节点为空。 getterConfig.xml 配置如下示例：

```
<?xml version="1.0" encoding="UTF-8"?>

<getterConfig>
    <getters>
        <getter sysOid="1.3.6.1.4.1.6339" name="DefaultSNMPGetter"></getter>
        <getter sysOid="1.3.6.1.4.1.63394" name="Huawei2300Getter"></getter>
    </getters>
</getterConfig>

```

sysOid: 设备的oid name: 采集的SNMP 采集器。

可选的采集器如下：

#### 思科

厂商|    采集器    |说明
---|------|---|
Cisco924Getter|    思科924型设备采集器
Cisco10700Getter    |思科10700型设备采集器
CiscoGetter    |思科设备通用采集器

#### 华三

厂商|    采集器    |说明
---|------|---|
H3cS10508Getter    |华三 S10508 型设备采集器
H3CGetter    |华三设备通用采集器

#### 华为

厂商|    采集器    |说明
---|------|---|
Huawei2300Getter    |华为2300 型设备采集器
Huawei3026cGetter    |华为3026c 型设备采集器
Huawei6500Getter    |华为 6500型设备采集器
Huawei6506Getter    |华为 6506 型设备采集器
Huawei8500Getter    |华为8500型设备采集器
Huawei9303Getter    |华为9303型设备采集器
HuaweiS2403hGetter    |华为S2403h 型设备采集器
HuaweiGetter|    华为设备通用采集器

### 输出结果说明

#### device

字段    |字段    |类型    |说明
----|-------|-------|----
id    |string|    设备id,唯一标识符|
centerX|    float    |设备在拓扑图中的 x 坐标
centerY    |float|    设备在拓扑图中的 y 坐标
ip    |string    |设备ip
devName    |string|    设备名称
devFirstType|    string|    设备一级类型
devSecondType|    string    |设备二级类型
oid    |string 设备系统| oid
hop|    int    |搜索深度|

#### link

字段    |字段    |类型    |说明
----|-------|-------|----
linkId    |string    |链路向量id
sx|    float    |链路向量起始 x 坐标
sy    |float|    链路向量起始 y 坐标
ex|    float    |链路向量结束 x 坐标
ey    |float    |链路向量结束 y 坐标
flinkId    |string|    设备到设备的id
deviceDestIfIdx    |string    |目标设备接口索引
deviceSrcIfIdx    |string|    源设备接口索引
deviceDestIfIdxName|    string|    目标设备接口名称
deviceSrcIfIdxName    |string|    源设备接口名称
startIp    |string    |源设备ip
endIp    |string    |目标设备ip
