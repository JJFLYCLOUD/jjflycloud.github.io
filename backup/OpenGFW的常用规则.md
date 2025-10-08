```
- name: '屏蔽连接25端口'
  action: block
  expr: proto == "tcp" && port.dst == 25

- name: '屏蔽连接中国IP的80端口'
  action: block
  expr: proto == "tcp"  &&  port.dst == 80  &&  geoip(string(ip.dst), "cn")
- name: '屏蔽连接中国IP的443端口'
  action: block
  expr: proto == "tcp"  &&  port.dst == 443  &&  geoip(string(ip.dst), "cn")

- name: '屏蔽来自中国IP的openvpn'
  action: block
  expr: openvpn != nil && openvpn.rx_pkt_cnt + openvpn.tx_pkt_cnt > 50 && geoip(string(ip.src), "cn")

- name: '屏蔽来自中国IP的wireguard通过handshake_response'
  action: drop
  expr: wireguard?.handshake_response?.receiver_index_matched == true && geoip(string(ip.src), "cn")

- name: '屏蔽socks5空密码'
  action: block
  expr: proto == "tcp" && socks?.req?.auth?.method == 0

- name: '屏蔽socks5空密码以及socks4协议'
  action: block
  expr: proto == "tcp" && socks?.version == 4

- name: '屏蔽来自中国IP的socks协议'
  action: block
  expr: proto == "tcp" && (socks != nil ) && geoip(string(ip.src), "cn")

- name: '屏蔽http代理'
  action: block
  expr: proto == "tcp" && http != nil && (string(http?.req?.path) matches "(^http://.*|.*:443$)")

- name: '屏蔽中国websocket'
  action: block
  expr: proto == "tcp" && http != nil && (string(http?.req?.headers.upgrade) == 'websocket' ) && geoip(string(ip.src), "cn")

- name: '屏蔽中国ss和vmess等全加密协议'
  action: block
  expr: proto == "tcp" && (fet != nil && fet.yes) && geoip(string(ip.src), "cn")
```

以上规则基本把易“土啬”的协议都屏蔽了，同时不影响海外使用这些协议，希望能帮助到你