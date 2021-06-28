# v2ray-VMess-TCP-HTTP-伪装
我们知道SSR 和v2ray 都可以免流, v2ray免流漏点更少 。v2ray采用tcp 和 http伪装方案在header 中加入混淆免流的网址来让计费系统误以为是免流数据流量。v2ray云免流脚本教程分为服务器端和客户端设置，需要有VPS主机服务器来实现v2ray服务器云免流，如果没有，也可用下文提到的注册免费BlueMix Docker容器，它和VPS主机用途相同。




1、搭建v2ray 免流服务器

v2ray 免流服务器端脚本配置如下：

{
	"log": {
		"loglevel": "warning"
	},
	"inbound": {
		"protocol": "VMess",
		"port": 8080,
		"settings": {
			"clients": [{
				"id": "ad806487-2d26-4636-98b6-ab85cc8521f7",
				"alterId": 64,
				"security": "chacha20-poly1305"
			}]
		},
		"streamSettings": {
			"network": "tcp",
			"httpSettings": {
				"path": "/"
			},
			"tcpSettings": {
				"header": {
					"type": "http",
					"response": {
						"version": "1.1",
						"status": "200",
						"reason": "OK",
						"headers": {
							"Content-Type": ["application/octet-stream", "application/x-msdownload", "text/html", "application/x-shockwave-flash"],
							"Transfer-Encoding": ["chunked"],
							"Connection": ["keep-alive"],
							"Pragma": "no-cache"
						}
					}
				}
			}
		}
	},
	"inboundDetour": [],
	"outbound": {
		"protocol": "freedom",
		"settings": {}
	}
}
2.v2ray 免流客户端配置

注意路径是/
根据我测试，手机的User-Agent和免流的Host只需在客户端设置，服务端无需设置。

如果你用v2rayN 软件，那配置如下

地址：就是你服务器IP地址或域名
端口：目前免流端口一般是80/8080/443，其它未测试
用户ID：填写服务器配置文件里的UUID
额外ID：填写服务器配置文件里的alterId
加密方式：不确定我选择的是none
传输协议：选择tcp
伪装类型：选择http伪装域名/其他项： 填写你手机卡的免流域名(我的是电信卡，所以填写: ltevod.tv189.cn )
底层传输安全：选择空

3. 实战例子——免费BlueMix Docker容器创建v2ray免流。

BlueMix Docker容器的使用可参考 免费BlueMix Docker容器免流 这篇文章。

3.1 设置BlueMix Docker

我们采用gingko/v2ray 这个docker镜像，环境变量CONFIG_JSON的参数填写上面服务器配置(用户ID要改成你自己的)。按下图填写。


4. v2ray 客服端配置——以v2ray Android 安卓客户端 BifrostV 来做演示

4.1、移动、联通、电信混淆参数设置。和SSR免流原理相同，v2ray也需要配置混淆host，各地运营商免流网址不同。目前电信/移动/联通三家的互联网套餐都可以实现v2ray 免流（云免），包括电信阿里鱼卡，联通腾讯王卡、蚂蚁宝卡、京东强卡、百度圣卡、网易白金卡、移动百度卡、芒果卡、天神卡等等，无需root，安卓、iOS通用免流方法。

三大运营商通常免流host（ v2ray免流头填写）：

v2ray免流移动混淆host：rd.go.10086.cn
v2ray免流联通混淆host：10001.cn
v2ray免流电信混淆host：ltevod.189.cn
4.2 免流安卓客户端

推荐BifrostV ，一个基于 v2ray 内核的 Android 应用，它支持 VMess、Shadowsocks、Socks 协议。它和SSR界面类似，简单易用，应用耗电量低。

BifrostV 破解版下载链接: https://pan.baidu.com/s/1IKjH9qUlAyY4_7mCXq357A 提取码: z4zt

按下图配置（可点击放大图片）：


经过以上配置即可实现安卓手机 v2ray云免流了。 或许你会问v2ray免流效果怎样？ 少许漏点，大概漏掉5%左右，因为v2ray支持udp免流。

如果是IOS苹果手机系统，可以使用 Kitsunebi ，Kitsunebi 是一个基于 v2ray 核心的 iOS 应用。Kitsunebi 支持导入和导出与 v2ray 兼容的 JSON 免流配置。

5、v2ray免流开热点

SSR免流可以通过注射器APP来开热点，那么v2ray免流可以开热点WIFI吗？理论上是可行的，如果是安卓手机root后安装注射器APP，然后开WIFI。

6、 v2ray路由器免流

Vray路由免流的原理和安卓手机端相同，使用路由器将数据加上http伪装的混淆host后，再发送给云服务器端。
拿openwrt搭建v2ray免流举例。安装v2ray后路由器配置如下：

{
  "log": {
    "error": "/tmp/syslog.log",
    "loglevel": "warning"
  },
  "inbound": {
    "port": 1088,
    "listen": "192.168.123.1",
    "protocol": "socks",
    "settings": {
      "auth": "noauth",
      "udp": true,
      "ip": "192.168.123.1"
    }
  },
  "inboundDetour": [
    {
      "port": "1099",
      "listen": "0.0.0.0",
      "protocol": "dokodemo-door",
      "settings": {
        "network": "tcp,udp",
        "timeout": 30,
        "followRedirect": true
      }
    }
  ],
  "outbound": {
    "protocol": "VMess",
    "settings": {
        "vnext": [
            {
                "address": "填你的服务器ip",
                "port": 填你的端口,
                "users": [
                    {
                        "id": "填你的uuid",
                        "alterId": 填你的alterid
                    }
                ]
            }
        ]
    },
    "streamSettings": {
      "network": "tcp",
      "tcpSettings": {
        "connectionReuse": true,
        "header": {
          "type": "http",
          "request": {
            "version": "1.1",
            "method": "GET",
            "path": ["/"],
            "headers": {
              "Host": ["你的免流host"],
              "User-Agent": [
                "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.75 Safari/537.36",
                        "Mozilla/5.0 (iPhone; CPU iPhone OS 10_0_2 like Mac OS X) AppleWebKit/601.1 (KHTML, like Gecko) CriOS/53.0.2785.109 Mobile/14A456 Safari/601.1.46"
              ],
              "Accept-Encoding": ["gzip, deflate"],
              "Connection": ["keep-alive"],
              "Pragma": "no-cache"
            }
          },
          "response": {
            "version": "1.1",
            "status": "200",
            "reason": "OK",
            "headers": {
              "Content-Type": ["application/octet-stream", "application/x-msdownload", "text/html", "application/x-shockwave-flash"],
              "Transfer-Encoding": ["chunked"],
              "Connection": ["keep-alive"],
              "Pragma": "no-cache"
            }
          }
        }
      }
    }
  },
  "outboundDetour": [
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    }
  ],
  "dns": {
    "servers": [
      "8.8.8.8",
      "8.8.4.4",
      "localhost"
    ]
  },
  "routing": {
    "strategy": "rules",
    "settings": {
      "domainStrategy": "IPIfNonMatch",
      "rules": []
    }
  }
}
