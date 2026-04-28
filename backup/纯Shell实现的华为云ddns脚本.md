

项目地址 https://github.com/JJFLYCLOUD/huaweicloud_ddns

fork自   https://github.com/lllvcs/huaweicloud_ddns

修改之处：v4和v6脚本二合一，自行更改注释行来切换；仅保留了在线查询公网IP，删除了通过网卡获取

使用方法：


```
#下载脚本
wget https://raw.githubusercontent.com/JJFLYCLOUD/huaweicloud_ddns/master/huaweicloud_ddns.sh
#赋予执行权限
chmod +x huaweicloud_ddns.sh
#放到不显眼的位置
mv huaweicloud_ddns.sh /etc/
#填入参数
vi /etc/huaweicloud_ddns.sh

ACCOUNTNAME=""  #账号中心里的'账号名'
USERNAME=""        #统一身份认证里的新建一个用户的用户名
PASSWORD=""  #统一身份认证里的新建一个用户的密码

创建一条A记录或者AAAA记录，然后点击修改这条解析(可以加个备注什么的)先别点确定，同时浏览器按F12打开调试，此时点击确定，调试框里应该会有一条ff开头的请求，查看这条请求的响应，zone_id填入ZONE_ID处(代表这个域名,同域名zone_id不会变)，id填入RECORDSET_ID(代表这条解析,每条解析对应一个id)

#手动执行
bash -x /etc/huaweicloud_ddns.sh
#添加定时任务
echo '* * * * * root bash /etc/huaweicloud_ddns.sh' >> /etc/crontab
```


华为DNS作为免费DNS里功能算是最强大的了，海外可以分大洲、分国家解析，国内可以分运营商、分省份解析；希望这个脚本能对你有帮助
