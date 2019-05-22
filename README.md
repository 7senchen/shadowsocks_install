Shadowsocks-go
wget --no-check-certificate -O shadowsocks-go.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-go.sh
chmod +x shadowsocks-go.sh
./shadowsocks-go.sh 2>&1 | tee shadowsocks-go.log

卸载：./shadowsocks-go.sh uninstall
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status
-------------------------------------------------
CentOS下shadowsocks-libev
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-libev.sh
chmod +x shadowsocks-libev.sh
./shadowsocks-libev.sh 2>&1 | tee shadowsocks-libev.log

卸载方法：
使用root用户登录Xshell，运行以下命令
./shadowsocks-libev.sh uninstall
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
查看状态：/etc/init.d/shadowsocks status
-----------------------------------------------------
OpenVZ 平台 Google BBR
https://blog.kuoruan.com/116.html
wget https://raw.githubusercontent.com/kuoruan/shell-scripts/master/ovz-bbr/ovz-bbr-installer.sh
chmod +x ovz-bbr-installer.sh
./ovz-bbr-installer.sh
需要配置的有如下几个选项：
1.需要加速的端口，即的 SS 端口。加速开启之后，流量会先经过 BBR 处理，之后再发送给后端的 SS。
2.可能需要配置 “公网接口名称”，即你服务器上具有公网 IP 的接口名称。搬瓦工 OpenVZ 上默认都是 venet0，但是有朋友可能需要安装在其他服务器上，
所以我加入了此选项。
需要注意的是，在有 firewalld 的服务器上安装的时候，firewalld 会干扰 iptables 的规则，造成网络不通（现在具体原因未知，谁有解决方案可以提示一下）。
所以在装有 firewalld 的服务器上需要先退出 firewalld：
systemctl disable firewalld
systemctl stop firewalld
如需卸载，请使用：
./ovz-bbr-installer.sh uninstall
多端口：
vi /usr/local/haproxy-lkl/etc/port-rules
使用 systemctl 或者 service 命令来启动、停止和重启 HAporxy-lkl：
systemctl {start|stop|restart} haproxy-lkl
service haproxy-lkl {start|stop|restart}
/usr/local/haproxy-lkl/etc/haproxy.cfg 这个文件是通过 port-rules 自动生成的，每次启动都会重新生成，所以直接修改它的配置没用。
如果想要自定义配置，请修改启动文件：
/usr/local/haproxy-lkl/sbin/haproxy-lkl
判断 bbr 是否正常启动可以尝试 ping 10.0.0.2，如果能通，说明 bbr 已经启动。
------------------------------------------
一键安装最新内核并开启 BBR 脚本
https://teddysun.com/489.html
本脚本适用环境
系统支持：CentOS 6+，Debian 7+，Ubuntu 12+
虚拟技术：OpenVZ 以外的，比如 KVM、Xen、VMware 等
内存要求：≥128M
日期　　：2018 年 12 月 14 日
使用root用户登录，运行以下命令：
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。
重启完成后，进入 VPS，验证一下是否成功安装最新内核并开启 TCP BBR，输入以下命令：
uname -r
查看内核版本，含有 4.11 就表示 OK 了
sysctl net.ipv4.tcp_available_congestion_control
返回值一般为：
net.ipv4.tcp_available_congestion_control = bbr cubic reno
sysctl net.ipv4.tcp_congestion_control
返回值一般为：
net.ipv4.tcp_congestion_control = bbr
sysctl net.core.default_qdisc
返回值一般为：
net.core.default_qdisc = fq
lsmod | grep bbr
返回值有 tcp_bbr 模块即说明bbr已启动。
内核升级方法
如果是 CentOS 系统，执行如下命令即可升级内核：
yum --enablerepo=elrepo-kernel -y install kernel-ml kernel-ml-devel
CentOS 6 的话，执行命令：
sed -i 's/^default=.*/default=0/g' /boot/grub/grub.conf
CentOS 7 的话，执行命令：
grub2-set-default 0
如果是 Debian/Ubuntu 系统，则需要手动下载最新版内核来安装升级。
去这里下载最新版的内核 deb 安装包。
如果系统是 64 位，则下载 amd64 的 linux-image 中含有 generic 这个 deb 包；
如果系统是 32 位，则下载 i386 的 linux-image 中含有 generic 这个 deb 包；
安装的命令如下（以最新版的 64 位 4.9.3 举例而已，请替换为下载好的 deb 包）：
dpkg -i linux-image-4.9.3-040903-generic_4.9.3-040903.201701120631_amd64.deb
安装完成后，再执行命令：
/usr/sbin/update-grub
最后，重启 VPS 即可。

如果你使用的是 Google Cloud Platform （GCP）更换内核，有时会遇到重启后，整个磁盘变为只读的情况。只需执行以下命令即可恢复：
mount -o remount rw /