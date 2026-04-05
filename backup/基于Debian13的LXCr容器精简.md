```
#设置自定义DNS
systemctl stop systemd-resolved
systemctl disable systemd-resolved
systemctl mask systemd-resolved
rm -f /etc/resolv.conf
echo 'nameserver 1.1.1.1
nameserver 8.8.8.8
nameserver 2606:4700:4700::1111
nameserver 2001:4860:4860::8888' > /etc/resolv.conf
```

```
#屏蔽systemd-journald日志输出，直接禁用会出错
echo "[Journal]
Storage=none" >/etc/systemd/journald.conf
systemctl restart systemd-journald
echo 'precedence ::ffff:0:0/96  100' > /etc/gai.conf
```

```
#apt安装需要的软件，并remove无用的软件包
apt install -y wget curl openssh-sftp-server openssh-server #建议都装上
apt install -y iperf3 vim jq cron #选装
apt remove -y bsd-mailx exim4-base exim4-config exim4-daemon-light polkitd pkexec xauth dialog krb5-locales net-tools udev sshpass isc-dhcp-client isc-dhcp-common
apt autoremove -y
```

```
#调整openssh配置文件，去除注释
echo 'Include /etc/ssh/sshd_config.d/*.conf
Port 22
PermitRootLogin yes
KbdInteractiveAuthentication no
UsePAM no
PrintMotd no
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/lib/openssh/sftp-server' > /etc/ssh/sshd_config
```

```
#关闭一些自动任务
systemctl stop apt-daily-upgrade.service apt-daily-upgrade.timer apt-daily.service apt-daily.timer dpkg-db-backup.service dpkg-db-backup.timer
systemctl disable apt-daily-upgrade.service apt-daily-upgrade.timer apt-daily.service apt-daily.timer dpkg-db-backup.service dpkg-db-backup.timer
rm -rf /usr/lib/systemd/system/apt-*
rm -rf /usr/lib/systemd/system/dpkg-*
```
```
#每天清理日志缓存，需要安装cron
echo '0 5 * * * root rm -rf /var/log/apt/*;rm -rf /var/cache/*;rm -rf  /var/log/*.log;rm -rf /var/log/wtmp' >>/etc/crontab
```
```
#帮助文档、手册、说明（最大头，最安全）
rm -rf /usr/share/man/*
rm -rf /usr/share/doc/*
rm -rf /usr/share/info/*

# 多语言翻译（你只用英文/中文，其他全冗余）
rm -rf /usr/share/locale/*

# 输入法、字体、键盘布局（容器无桌面，无用）
rm -rf /usr/share/fonts/*
rm -rf /usr/share/ibus*
rm -rf /usr/share/keymaps
rm -rf /usr/share/consolefonts

# 游戏、屏幕保护、壁纸（纯垃圾）
rm -rf /usr/share/games
rm -rf /usr/share/screensavers
rm -rf /usr/share/backgrounds

# 桌面无关的图标、主题
rm -rf /usr/share/icons
rm -rf /usr/share/themes

# 帮助中心、示例、教程
rm -rf /usr/share/help
rm -rf /usr/share/examples
rm -rf /usr/share/gettext
rm -rf /usr/share/doc-base
rm -rf /usr/share/zoneinfo-leaps
rm -rf /usr/share/vim/vim90/doc

#清理环境
apt clean;apt autoremove -y;apt autoclean;rm -rf /var/log/apt/*;rm -rf /var/cache/*;rm -rf  /var/log/*.log;rm -rf /var/log/wtmp
rm /var/lib/dhcp/*
rm  /var/log/*.log
rm  /var/log/wtmp
rm /var/log/apt/*
rm  -rf tmp/*
rm .bash_history
history -c
```


当然更建议容器使用alpine系统
