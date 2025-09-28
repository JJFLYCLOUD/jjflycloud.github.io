### Alpine 添加服务(类似systemd)

之前一直在用Debian系统来部署应用等，使用systemd给程序保活、开机自启比较方便；如果VPS配置低，Alpine可以比Debian可以节省出更多的资源

Alpine一般使用openrc来管理服务
配置文件在 `/etc/init.d/`目录
对于已经编译好无依赖的二进制程序(比如哪吒探针客户端)，示例如下

`vi /etc/init.d/nezha-agent #编辑文件`

添加以下内容，具体需要根据实际来更改

```
#!/sbin/openrc-run

name="nezha-agent" #服务名
supervisor="supervise-daemon" 
command="/usr/sbin/nezha-agent" #二进制文件的路径
command_args="-s 127.0.0.1:5555 -p 114514 --disable-force-update --disable-auto-update  --skip-conn --skip-procs" #执行参数
directory="/opt/nezha" #运行目录
extra_started_commands="" #服务启动以后执行的命令
extra_stopped_commands="" #服务停止以后执行的命令
command_user="root:root" #执行用户和用户组

#在网络启动后启动服务，二进制程序一般无需改动
depend() {
        after net dns
        use net
}
```

保存文件后

```
chmod +x /etc/init.d/nezha-agent #给可执行权限
rc-update add nezha-agent #添加服务，开机自启，相当于的systemctl enable nezha-agent
rc-update del nezha-agent  #删除服务，取消自启，相当于的systemctl disable nezha-agent
rc-service start      nezha-agent #启动服务，相当于systemctl start nezha-agent
rc-service restart  nezha-agent #重启服务，相当于systemctl restart nezha-agent
rc-service stop      nezha-agent #关闭服务，相当于systemctl stop nezha-agent
rc-status #运行状态，可以看到所有服务的状态
```

以上就是二进制无依赖的程序如何在Alpine上实现开机自启、进程保活，希望能帮助到你

