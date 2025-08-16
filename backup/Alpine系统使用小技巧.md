### Alpine 使用技巧

之前一直在用Debian系统来部署应用等，使用systemd给程序保活、开机自启比较方便，虽然Debian系统本身占用不高，但对比Alpine还是小巫见大巫了
现在市面上有很多拼好鸡，使用Alpine可以省出来更多的内存和硬盘

Alpine一般使用openrc来管理服务
配置文件在 `/etc/init.d/`目录
对于已经编译好无依赖的二进制程序(比如哪吒探针客户端)，示例如下

`nano /etc/init.d/nezha-agent`

```
name="nezha-agent" #服务名
supervisor="supervise-daemon" 
command="/usr/sbin/nezha-agent" #二进制文件的路径
command_args="-s 127.0.0.1:5555 -p 114514 --disable-force-update --disable-auto-update  --skip-conn --skip-procs" #执行参数
extra_started_commands="" #服务启动以后执行的命令
extra_stopped_commands="" #服务停止以后执行的命令
command_user="root:root" #执行用户和用户组

#在网络启动后启动服务，二进制程序一般无需改动
depend() {
        after net dns
        use net
}
#保存文件后
chmod +x /etc/init.d/nezha-agent #给可执行权限
rc-update add nezha-agent #添加服务，开机自启，相当于Debian下的systemctl enable nezha-agent
rc-service start nezha-agent #启动服务，相当于Debian下的systemctl start nezha-agent
#之后这个程序就可以随系统启动了top命令里也可以看到该命令在执行中
"/usr/sbin/nezha-agent -s 127.0.0.1:5555 -p 114514 --disable-force-update --disable-auto-update  --skip-conn --skip-procs"

rc-update del nezha-agent #取消开机自启，相当于Debian下的systemctl disable nezha-agent
rc-service restart  nezha-agent #重启服务 ，相当于Debian下的systemctl restart nezha-agent
rc-service stop  nezha-agent #关闭服务，相当于Debian下的systemctl stop nezha-agent
```

以上就是二进制无依赖的程序如何在Alpine上实现开机自启、进程保活，希望能帮助到你
