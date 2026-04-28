
* **[dlundquist/sniproxy](https://github.com/dlundquist/sniproxy)** 老项目，工作稳定，linux 软件包一般自带，可以通过 (`Debian` 系 `apt install sniproxy`，`Alpine` 可以 `apk add sniproxy`)。分流配置方便，缺点就是太老了，对于双栈机器配置摸不到头脑。
* **[XIU2/SNIProxy](https://github.com/XIU2/SNIProxy)** 新项目，go 编写，不挑平台，可以再套 socks出口，缺点就是可靠性有待验证，以及分流不方便。

---

后来发现通过透明代理中的 `redirect` 也可以实现此功能，`xray`、`sing-box`、`gost` 等有redirect功能

### 以 sing-box 配置举例

```json
{
  "log": {
    "level": "panic"
  },
  "inbounds": [
    // 监听 80 443 端口
    {
      "type": "redirect",
      "tag": "sniproxy_80",
      "listen": "::",
      "listen_port": 80
    },
    {
      "type": "redirect",
      "tag": "sniproxy_443",
      "listen": "::",
      "listen_port": 443
    }
  ],
  "dns": {
    // DNS 配置, 可以手动设置 v4 优先出站
    "strategy": "prefer_ipv4",
    "servers": [
      {
        "type": "local",
        "tag": "local",
        // 预定义域名，假如从 80 443 端口传来 domain1.com 的数据，会解析为 1.1.1.1 并访问
        "predefined": {
          "domain1.com": "1.1.1.1"
        }
      }
    ]
  },
  "outbounds": [
    // 设置出栈，可以手动设置 v4 v6 优先，甚至可以再套 WARP 或者什么其他协议出栈，很方便
    {
      "type": "direct",
      "tag": "direct-v4",
      "domain_resolver": {
        "server": "local",
        "strategy": "prefer_ipv4"
      }
    },
    {
      "type": "direct",
      "tag": "direct-v6",
      "domain_resolver": {
        "server": "local",
        "strategy": "prefer_ipv6"
      }
    }
  ],
  "route": {
    "final": "direct-v4", // 默认出栈，即下方规则没匹配到的域名通过此出栈
    "rules": [
      // 域名嗅探，这是关键配置，否则无法识别域名
      {
        "inbound": "sniproxy_80",
        "action": "sniff",
        "timeout": "1s"
      },
      {
        "inbound": "sniproxy_443",
        "action": "sniff",
        "timeout": "1s"
      },
      // 分流规则，可以设置指定域名优先通过 v4 或 v6 访问
      // PS：以下规则可以防止IPv4地址 google 送中
      {
        "domain_suffix": [
          "youtube.com",
          "google.com"
        ],
        "domain_keyword": [
          "pa.googleapis.com"
        ],
        "action": "route",
        "outbound": "direct-v4"
      },
      {
        "domain_suffix": [
          "googleapis.com",
          "googleapis.cn"
        ],
        "action": "route",
        "outbound": "direct-v6"
      }
    ]
  }
}
```

