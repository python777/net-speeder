# net-speeder
net-speeder 在高延迟不稳定链路上优化单线程下载速度 

项目由https://code.google.com/p/net-speeder/  迁入


A program to speed up single thread download upon long delay and unstable network

在高延迟不稳定链路上优化单线程下载速度

注1：开启了net-speeder的服务器上对外ping时看到的是4倍，实际网络上是2倍流量。另外两倍是内部dup出来的，不占用带宽。
另外，内部dup包并非是偷懒未判断。。。是为了更快触发快速重传的。
注2：net-speeder不依赖ttl的大小，ttl的大小跟流量无比例关系。不存在windows的ttl大，发包就多的情况。

运行时依赖的库：libnet， libpcap

debian/ubuntu安装libnet：apt-get install libnet1

    安装libpcap： apt-get install libpcap0.8 

编译需要安装libnet和libpcap对应的dev包 debian/ubuntu安装libnet-dev：apt-get install libnet1-dev

    安装libpcap-dev： apt-get install libpcap0.8-dev 

centos安装： 下载epel：https://fedoraproject.org/wiki/EPEL/zh-cn 例：CentOS6 64位：

wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

如果是centos5，则在epel/5/下

然后安装epel：rpm -ivh epel-release-X-Y.noarch.rpm

然后即可使用yum安装：yum install libnet libpcap libnet-devel libpcap-devel

编译：

Linux Cooked interface使用编译（venetX，OpenVZ）： sh build.sh -DCOOKED 已测试

普通网卡使用编译（Xen，KVM，物理机）： sh build.sh 待测试

使用方法(需要root权限启动）：

参数：./net_speeder 网卡名 加速规则（bpf规则）

最简单用法： # ./net_speeder venet0 "ip" 加速所有ip协议数据。

##
##

以下是高级点的用法：
 net-speeder的bpf规则 bpf rules for net-speeder
net-speeder 的使用方法为

    ./net_speeder 网卡名 加速规则

其中加速规则采用 bpf 规则，一般常用下面规则来启动

    ./net_speeder venet0 "ip" 

    ./net_speeder venet0 "tcp"

这样可能存在安全隐患，可用 iptables 来处理，但并未从根源来解决
https://www.v2ex.com/t/248273 

其实可以用 bpf 规则使之只复制发出的包，下面 3 条规则供参考
        复制本机发出的 tcp 包：

    ./net_speeder venet0 "tcp and src host 本机 IP 地址" 

        复制本机某个端口发出的 tcp 包：

    ./net_speeder venet0 "tcp src port 端口号 and src host 本机 IP 地址"

        复制本机多个端口发出的 tcp 包：

    ./net_speeder venet0 "(tcp src port 端口号1 or 端口号2 or 端口号3) and src host 本机 IP 地址" 
    
    （注：本机 IP 地址不要用 127.0.0.1 ，可用 hostname -I | tr ' ' '\n' | grep '[0-9]' | grep -v '127\.0\.0\.'来获取）
    
    via： http://youdemeide.blogspot.com/2016/01/net-speeder.html
