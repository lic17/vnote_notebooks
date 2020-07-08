# 1. linux网络虚拟化

## 1.1. network namespace

### 1.1.1. 初识network namespace
创建network namespace: netns1

 ```
 # ip netns add netns1
 ```

进入netns1执行ip link list命令

 ```
 # ip netns exec netns1 ip link list
 ```

查看系统中的network namespace

 ```
 # ip netns list
 ```
删除network namespace

```
# ip netns delete netns1
```

### 1.1.2. 配置network namespace

进入netns1 ping 127.0.0.1失败需要将lo设备启动

```
sudo ip netns exec netns1 ping 127.0.0.1
ping: connect: 网络不可达
```

进入netns1将lo设备状态设置成UP，启动lo设备

```
# 2. ip netns exec netns1 ip link set dev lo up
```
再次ping 127.0.0.1成功

```
sudo ip netns exec netns1 ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.032 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.056 ms
^C
--- 127.0.0.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.032/0.044/0.056/0.012 ms

```

#### 1.1.2.1. 配置network namespace与外界通信

创建一对虚拟以太网卡，并把veth pair一端放到netns1中
```
sudo ip link add veth0 type veth peer name veth1
sudo ip link set veth1 netns netns1
```

为以太网卡对配置ip并启动
```
sudo ip netns exec netns1 ifconfig veth1 10.1.1.1/24 up
sudo ifconfig veth0 10.1.1.2/24 up
```
ping netns1内部网卡veth1 IP:10.1.1.1
```
ping 10.1.1.1
PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
64 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=0.120 ms
64 bytes from 10.1.1.1: icmp_seq=2 ttl=64 time=0.089 ms
^C
--- 10.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1026ms
rtt min/avg/max/mdev = 0.089/0.104/0.120/0.015 ms
```
从netns1内部ping外部网卡veth0 IP:10.1.1.2
```
sudo ip netns exec netns1 ping 10.1.1.2
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.038 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.085 ms
^C
--- 10.1.1.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1015ms
rtt min/avg/max/mdev = 0.038/0.061/0.085/0.023 ms
```

#### 1.1.2.2. 根据PID进行namespace迁移
进入netns1， 把netns1下veth1网卡挪到PID为1的进程所在的network namespace中

```
ip netns exec netns1 ip link set veth1 netns 1
```

## 1.2. veth pair
## 1.3. linux bridge
## 1.4. tun/tap 设备
## 1.5. iptables
## 1.6. linux隧道 ipip
## 1.7. VXLAN
## 1.8. Macvlan
## 1.9. IPvlan
## 1.10 进入容器 ns
获取容器的进程号
```
docker inspect 容器|grep Pid
```
执行如下命令，将进程网络命名空间恢复到主机目录，

```
ln -s /proc/容器进程号/ns/net /var/run/netns/容器
```
如果/var/run/netns目录不存在，以root用户手动创建目录即可。
然后执行ip netns命令即可看到容器的网络命名空间。可执行网络相关命令来排查网络情况。