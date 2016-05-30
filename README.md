# Performance of Docker Overlay Network

## 摘要
Docker1.9+提供覆盖网络(overlay)，用于支持夸主机通信。本文选取overlay网络模式和原生ovs网络模式，考虑CPU资源消耗、网络吞吐量和延迟三个度量指标，对比容器夸主机通信的性能。

## 实验设计 

#### 测试环境

- 两台ThinkCenter M8000+主机，配置信息见下表(待补充)

Spec |  Matrix
				----|------|
				Environment | Dell OPTIPLEX 9010
				Software | iperf3 && nuttcp
				CPU |  4 Core x Intel(R) Core(TM) i5-3470 CPU @ 3.20GHz
				Memory | 8G 
				Nic |  100Mb/s
				Operating System | CentOS Linux 7.1.1503 (Core)
				Kernel | 3.10.0-229.4.2.el7.x86_64

- Centos7.1
- Docker1.11
- 容器为busybox

#### 测试工具

- 网络延迟和吞吐量 ：iperf
- CPU 资源消耗 : top

#### 测试用例

单节点测试
- overlay模式下：两个容器之间
- overlay模式下：容器和宿主机之间
- ovs模式下：两个容器之间
- OVS模式下：容器和宿主机之间

夸主机测试
- overlay模式下：宿主机之间
- overlay模式下：夸主机两个容器之间
- ovs模式下：宿主机之间
- ovs模式下：跨主机两个容器之间

采用以下方法避免测试结果偶然性

- 对称安装： 对于每个测试用例，节点1安装iperf client， 节点2安装iperf server；节点2安装iperf server, 节点1安装iperf client， 两次测试结果取平均
- 多次测试取平均值：每个测试用例运行*5次*，对测试结果求平均值

## 实验安装

- iperf安装
- [overlay网络安装](https://github.com/oncecloud/testing/blob/master/overlay-install.md)
- [ovs网络安装](https://github.com/oncecloud/testing/blob/master/ovs-install.md)

## 实验结果

#### 原生(两台物理主机之间)

吞吐量

- Node1安装iperf server， Node2安装iperf client

```
[  5] local 133.133.134.153 port 5201 connected to 133.133.134.131 port 43321
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec   107 MBytes   901 Mbits/sec                  
[  5]   1.00-2.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   2.00-3.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   3.00-4.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   4.00-5.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   5.00-6.00   sec   112 MBytes   936 Mbits/sec                  
[  5]   6.00-7.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   7.00-8.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   8.00-9.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   9.00-10.00  sec   112 MBytes   937 Mbits/sec                  
[  5]  10.00-10.04  sec  4.41 MBytes   933 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  5]   0.00-10.04  sec  1.09 GBytes   934 Mbits/sec    0             sender
[  5]   0.00-10.04  sec  1.09 GBytes   933 Mbits/sec                  receiver

```

- Node1安装iperf client, Node2安装iperf server

```
[  5] local 133.133.134.131 port 5201 connected to 133.133.134.153 port 34102
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec   107 MBytes   902 Mbits/sec                  
[  5]   1.00-2.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   2.00-3.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   3.00-4.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   4.00-5.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   5.00-6.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   6.00-7.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   7.00-8.00   sec   112 MBytes   937 Mbits/sec                  
[  5]   8.00-9.00   sec   112 MBytes   936 Mbits/sec                  
[  5]   9.00-10.00  sec   112 MBytes   937 Mbits/sec                  
[  5]  10.00-10.04  sec  4.41 MBytes   932 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  5]   0.00-10.04  sec  1.09 GBytes   934 Mbits/sec    0             sender
[  5]   0.00-10.04  sec  1.09 GBytes   933 Mbits/sec                  receiver
```

延迟
- Node1 ping Node2

```
PING 133.133.134.131 (133.133.134.131) 56(84) bytes of data.
64 bytes from 133.133.134.131: icmp_seq=1 ttl=64 time=0.142 ms
64 bytes from 133.133.134.131: icmp_seq=2 ttl=64 time=0.088 ms
64 bytes from 133.133.134.131: icmp_seq=3 ttl=64 time=0.125 ms
64 bytes from 133.133.134.131: icmp_seq=4 ttl=64 time=0.119 ms
64 bytes from 133.133.134.131: icmp_seq=5 ttl=64 time=0.088 ms

--- 133.133.134.131 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 3999ms
rtt min/avg/max/mdev = 0.088/0.112/0.142/0.023 ms
```

#### overlay

- Container1安装iperf client, Container2安装iperf server

```
[  5] local 10.0.0.2 port 5201 connected to 10.0.0.3 port 56712
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec   103 MBytes   861 Mbits/sec                  
[  5]   1.00-2.00   sec   108 MBytes   904 Mbits/sec                  
[  5]   2.00-3.00   sec   108 MBytes   904 Mbits/sec                  
[  5]   3.00-4.00   sec   108 MBytes   904 Mbits/sec                  
[  5]   4.00-5.00   sec   108 MBytes   904 Mbits/sec                  
[  5]   5.00-6.00   sec   108 MBytes   904 Mbits/sec                  
[  5]   6.00-7.00   sec   108 MBytes   904 Mbits/sec                  
[  5]   7.00-8.00   sec   108 MBytes   904 Mbits/sec                  
[  5]   8.00-9.00   sec   108 MBytes   904 Mbits/sec                  
[  5]   9.00-10.00  sec   108 MBytes   904 Mbits/sec                  
[  5]  10.00-10.06  sec  6.84 MBytes   904 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  5]   0.00-10.06  sec  1.05 GBytes   900 Mbits/sec    0             sender
[  5]   0.00-10.06  sec  1.05 GBytes   900 Mbits/sec                  receiver

```
- Container1 ping Container2

```
PING c1 (10.0.0.2) 56(84) bytes of data.
64 bytes from c1.ovr0 (10.0.0.2): icmp_seq=1 ttl=64 time=0.215 ms
64 bytes from c1.ovr0 (10.0.0.2): icmp_seq=2 ttl=64 time=0.185 ms
64 bytes from c1.ovr0 (10.0.0.2): icmp_seq=3 ttl=64 time=0.195 ms
64 bytes from c1.ovr0 (10.0.0.2): icmp_seq=4 ttl=64 time=0.240 ms
64 bytes from c1.ovr0 (10.0.0.2): icmp_seq=5 ttl=64 time=0.198 ms

--- c1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4000ms
rtt min/avg/max/mdev = 0.185/0.206/0.240/0.024 ms
```



#### ovs

- Container1安装iperf client, Container2安装iperf server

```
```
- Container1安装iperf server, Container2安装iperf client

```
```
