#### KAlIの一些网络操作

1. arping -c 192.168.10.1 缺点是只能对单一ip进行探测，可利用shell脚本进行网段探测扫描

1. netdiscover -i eth0 -r 192.168.10.0/24 主动方式


- netdiscover -p 以被动方式扫描局域网下所有存活主机

1. fping -g 192.168.10.0/24 -c 1 > 1.txt

1. hping3 -c 1000 -d 120 -S -w 64 -p 80 --flood --rand-source baidu.com 类似洪水攻击


1. Nmap：
   - nmap -sn 192.168.10.0/24    -sn只进行ping扫描
   - nmap -sS ip -p 80,443,22,21 对目标ip进行半连接扫描哪些端口开放，-sS表示半连接扫描

nc (netcat) ：
作用：
1.实现任意tcp/udp端口的监听，nc可作为server以tcp或udp方式监听指定端口
2.端口扫描，nc可作为client发起tcp/udp连接
3.机器之间传输文件
4.机器之间网络测速
参数：
-nv 表示要扫描的目标是一个ip，不做域名解析
-w 表示超时时间
-z 表示进行端口扫描
实例：
nc -nv -w 1 -z 123.57.106.233 1-60053

Scapy:
使用scapy进行定制数据包高级扫描，scapy是一个可以让用户发送，侦听和解析并伪装网络报文的python程序。
1.定制arp协议
ARP协议的标准格式：
 hwtype= 0x1	硬件类型
  ptype= IPv4	协议类型
  hwlen= None	硬件地址长度（mac）
  plen= None	协议地址长度（ip）
  op= who-has	who-has查询
  hwsrc= 00:0c:29:59:3a:7a	源mac地址
  psrc= 192.168.10.53	源ip地址
  hwdst= 00:00:00:00:00:00
  pdst= 0.0.0.0	向谁发送查询请求

例：定义向192.168.10.1发送arp请求的数据包
sr1函数：包含了发送和接收数据包的功能
sr1(ARP(pdst="192.168.10.1"))

定制IP包：
标准格式：
version= 4	IPV4版本
  ihl= None	首部长度
  tos= 0x0	服务
  len= None	总长度
  id= 1	标识
  flags= 	
  frag= 0	标志
  ttl= 64	生存时间
  proto= hopopt	传输控制协议，IPV6逐跳选项
  chksum= None	首部校验和
  src= 127.0.0.1	源地址
  dst= 127.0.0.1	目标地址
  \options\

定制ICMP包：
ICMP数据包标准格式：
type= echo-request	类型，标识ICMP报文的类型request/reply
  code= 0			代码
  chksum= None		检验和
  id= 0x0			标识
  seq= 0x0

注明：IP()生成ping包的源IP和目标IP，ICMP()生成ping包的类型。
使用IP(),ICMP()两个函数，可以生成ping包，进行探测。
例:
sr1(IP(dst="192.168.10.11")/ICMP(),timeout=1)

定制TCP协议是SYN数据包：
标准格式：
sport= ftp_data	tcp源端口
  dport= http	目标端口
  seq= 0		32位序号
  ack= 0		32位确认序号
  dataofs= None	4位首部长度
  reserved= 0	保留6位
  flags= S		标志域，紧急标志，有意义的应答标志，推，重置连接标志，同步序列号标志，完成发送数据标志。
按照排列顺序是：URG, ACK, PSH, RST, SYN. FIN
  window= 8192	窗口大小
  chksum= None	16位校验和
  urgptr= 0		优先指针
  options= []		选项

例：
sr1(IP(dst="192.168.10.11")/TCP(flags="S",dport=80),timeout=1)
有回复说明80端口开放，回应中，flags="S"表示SYN数据包

僵尸扫描（肉鸡）：
nmap 192.168.10.0/24 -p1-1024 --script=ipidseq.nse
指定主机代理扫描：
nmap 192.168.10.11 -sI 192.168.10.22 -p1-100
其中11为目标主机，22为目标代理扫描主机