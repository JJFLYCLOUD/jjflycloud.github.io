TLS证书有效期越来越短，自动化越来越有必要，我一般使用acme.sh，项目地址
[https://github.com/acmesh-official/acme.sh/wiki/说明](https://github.com/acmesh-official/acme.sh/wiki/说明)

看重它的只有一点：安装过程不会污染已有的系统任何功能和文件，所有的修改都限制在安装目录中: `~/.acme.sh/` 删除或者备份都非常方便，只需要把`~/.acme.sh/`目录打包即可，再手动加一条定时任务

## 安装

`curl https://get.acme.sh | sh -s email=my@example.com （邮箱建议用真实的，证书变动可以收到提醒）`


## 常用命令

`acme.sh --list  显示域名列表`
`acme.sh --remove -d a.com  删除域名的自动续签，但不会删除文件`
`acme.sh --renew -d a.com 手动续签a.com证书，加 --force强制续签`
`acme.sh --renew-all 	 手动续签所有证书，加 --force强制续签`
`acme.sh --set-default-ca --server letsencrypt #设置LETS为默认证书提供商，具体server查看` `https://github.com/acmesh-official/acme.sh/wiki/Server`
`acme.sh  --upgrade  --auto-upgrade  设置自动升级`

## 常用的几种证书申请方式

* HTTP验证

`acme.sh --issue -d a.com -d b.com -d c.com --webroot /home/wwwroot/mydomain.com/`
　　补充：可以一次性验证多个域名，即使二级域名不一样也可以，无法签发泛域名；多个域名只生成一组tls证书；缺点就是国内服务器必须备案才能用，甚至例如阿里云的服务器等必须在阿里云备案才能用。
　　acme还有个独立服务模式，如果80端口是空闲的可以用此命令，我一般都是和nginx混用，没用过这个命令
`acme.sh --issue --standalone -d example.com -d www.example.com -d cp.example.com`，如果设置了自动跳转https，需要对\.well-known单独处理，贴上我的nginx配置：

```
server
{
listen 80 default_server;
#listen [::]:80 fastopen=3;
root /www/wwwroot/proxy.com;
location ~ \.well-known{
allow all;
}
location / {
return 301 https://$host$request_uri;
}
access_log /dev/null;
error_log /dev/null;
}
```

* DNS验证

`acme.sh --issue --dns dns_cf   -d a.com -d *.a.com`
补充：可以签发泛解析域名，

## 花活

* 如果你想一次性申请多个域名的证书，但这些证书托管在不同的DNS服务商，可以用cname的方式授权某个域名来进行DNS验证
  官方文档在这里  [https://github.com/acmesh-official/acme.sh/wiki/DNS-alias-mode](https://github.com/acmesh-official/acme.sh/wiki/DNS-alias-mode)
  举个例子
  通过a.com(有DNS的API，且已托管到CF) 来签发 a.b.com(无DNS的API) 的证书
  先添加解析在a.b.com域名添加一条解析：
  `_acme-challenge.a.b.com    cname 指向   _acme-challenge.a.com`
  之后执行
  `acme.sh --issue -d  a.b.com -d *a.b.com  --challenge-alias a.com --dns dns_cf`

## 添加DNS的API

官方的的API导入方法是用export
我一般直接写在`~/.acme.sh/account.conf`里，方便以后迁移
我用过的API参数

```
#华为云
SAVED_HUAWEICLOUD_Username=''
SAVED_HUAWEICLOUD_Password=''
SAVED_HUAWEICLOUD_DomainName=''
```

```
#CloudFlare
CF_Token=""
CF_Account_ID=""
CF_Zone_ID=""
```

## 安装证书

#宝塔的nginx重载命令是`nginx -s reload`，LNMP可能是`systemctl reload nginx`
`acme.sh --install-cert -d a.com --key-file /path/key.pem  --fullchain-file /path/fullchain.pem --reloadcmd "nginx -s reload"`
`--cert-file`一般用不到，可以不安装
