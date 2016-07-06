#Comparsion of Native, Bridge, OVS Network Mode of Docker

----------

# BenchMark Spec

				Spec |  Matrix
				----|------|
				Environment | Dell OPTIPLEX 9010
				Software | iperf3 && nuttcp
				CPU |  4 Core x Intel(R) Core(TM) i5-3470 CPU @ 3.20GHz
				Memory | 8G 
				Nic |  100Mb/s
				Operating System | CentOS Linux 7.1.1503 (Core)
				Kernel | 3.10.0-229.4.2.el7.x86_64


# Native

- Native模式下：容器安装iPerf server，物理主机安装iPerf client（测试脚本1） 

	启动参数

		docker run -it --rm --net=host centos:7.1.1503 /bin/bash

	执行结果 

		Connecting to host 133.133.134.71, port 5201
		[  4] local 133.133.134.71 port 45355 connected to 133.133.134.71 port 5201
		[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
		[  4]   0.00-1.00   sec  5.89 GBytes  50.6 Gbits/sec    0    959 KBytes       
		[  4]   1.00-2.00   sec  6.07 GBytes  52.2 Gbits/sec    0   1.06 MBytes       
		[  4]   2.00-3.00   sec  6.10 GBytes  52.4 Gbits/sec    0   1.06 MBytes       
		[  4]   3.00-4.00   sec  6.12 GBytes  52.6 Gbits/sec    0   1.06 MBytes       
		[  4]   4.00-5.00   sec  6.15 GBytes  52.8 Gbits/sec    0   1.06 MBytes       
		[  4]   5.00-6.00   sec  6.15 GBytes  52.8 Gbits/sec    0   1.06 MBytes       
		[  4]   6.00-7.00   sec  6.10 GBytes  52.4 Gbits/sec    0   1.06 MBytes       
		[  4]   7.00-8.00   sec  6.16 GBytes  52.9 Gbits/sec    0   1.06 MBytes       
		[  4]   8.00-9.00   sec  6.16 GBytes  52.9 Gbits/sec    0   1.06 MBytes       
		[  4]   9.00-10.00  sec  6.16 GBytes  52.9 Gbits/sec    0   1.06 MBytes       
		- - - - - - - - - - - - - - - - - - - - - - - - -
		[ ID] Interval           Transfer     Bandwidth       Retr
		[  4]   0.00-10.00  sec  61.1 GBytes  52.5 Gbits/sec    0             sender
		[  4]   0.00-10.00  sec  61.1 GBytes  52.5 Gbits/sec                  receiver
		
		iperf Done.

- Native模式下：容器安装iPerf client，物理主机安装iPerf server（测试脚本1）  


	启动参数

		docker run -it --rm --net=host centos:7.1.1503 /bin/bash

	执行结果 

		Connecting to host 133.133.134.71, port 5201
		[  4] local 133.133.134.71 port 45367 connected to 133.133.134.71 port 5201
		[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
		[  4]   0.00-1.00   sec  5.97 GBytes  51.3 Gbits/sec    0   1.06 MBytes       
		[  4]   1.00-2.00   sec  6.16 GBytes  52.9 Gbits/sec    0   1.06 MBytes       
		[  4]   2.00-3.00   sec  6.17 GBytes  53.0 Gbits/sec    0   1.06 MBytes       
		[  4]   3.00-4.00   sec  6.15 GBytes  52.8 Gbits/sec    0   1.06 MBytes       
		[  4]   4.00-5.00   sec  6.18 GBytes  53.0 Gbits/sec    0   1.06 MBytes       
		[  4]   5.00-6.00   sec  6.18 GBytes  53.1 Gbits/sec    0   1.06 MBytes       
		[  4]   6.00-7.00   sec  6.19 GBytes  53.2 Gbits/sec    0   1.06 MBytes       
		[  4]   7.00-8.00   sec  6.15 GBytes  52.9 Gbits/sec    0   1.06 MBytes       
		[  4]   8.00-9.00   sec  6.17 GBytes  53.0 Gbits/sec    0   1.06 MBytes       
		[  4]   9.00-10.00  sec  6.17 GBytes  53.0 Gbits/sec    0   1.06 MBytes       
		- - - - - - - - - - - - - - - - - - - - - - - - -
		[ ID] Interval           Transfer     Bandwidth       Retr
		[  4]   0.00-10.00  sec  61.5 GBytes  52.8 Gbits/sec    0             sender
		[  4]   0.00-10.00  sec  61.5 GBytes  52.8 Gbits/sec                  receiver
		
		iperf Done.

- Native模式下：两个容器分别安装iPerf server和iPerf client（测试脚本1）  


	启动参数
		
		执行以下命令创建两个容器
		docker run -it --rm --net=host centos:7.1.1503 /bin/bash

	执行结果 

		Connecting to host 133.133.134.71, port 5201
		[  4] local 133.133.134.71 port 45378 connected to 133.133.134.71 port 5201
		[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
		[  4]   0.00-1.00   sec  5.99 GBytes  51.5 Gbits/sec    0   1023 KBytes       
		[  4]   1.00-2.00   sec  6.15 GBytes  52.8 Gbits/sec    0   1023 KBytes       
		[  4]   2.00-3.00   sec  6.15 GBytes  52.8 Gbits/sec    0   1023 KBytes       
		[  4]   3.00-4.00   sec  6.16 GBytes  52.9 Gbits/sec    0   1023 KBytes       
		[  4]   4.00-5.00   sec  6.09 GBytes  52.4 Gbits/sec    0   1023 KBytes       
		[  4]   5.00-6.00   sec  6.08 GBytes  52.3 Gbits/sec    0   1023 KBytes       
		[  4]   6.00-7.00   sec  6.14 GBytes  52.7 Gbits/sec    0   1023 KBytes       
		[  4]   7.00-8.00   sec  6.13 GBytes  52.7 Gbits/sec    0   1.06 MBytes       
		[  4]   8.00-9.00   sec  6.18 GBytes  53.1 Gbits/sec    0   1.06 MBytes       
		[  4]   9.00-10.00  sec  6.17 GBytes  53.0 Gbits/sec    0   1.06 MBytes       
		- - - - - - - - - - - - - - - - - - - - - - - - -
		[ ID] Interval           Transfer     Bandwidth       Retr
		[  4]   0.00-10.00  sec  61.2 GBytes  52.6 Gbits/sec    0             sender
		[  4]   0.00-10.00  sec  61.2 GBytes  52.6 Gbits/sec                  receiver
		
		iperf Done.

- Native模式下：容器安装Nuttcp server，物理主机安装Nuttcp client（测试脚本2）  

	启动参数

		docker run -it --rm --net=host centos:7.1.1503 /bin/bash

	执行结果 

		[root@master ~]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   4.01 sec = 21401.1338 Mbps 99 %TX 86 %RX 0 retrans 0.08 msRTT
		[root@master ~]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.88 sec = 22135.0164 Mbps 99 %TX 87 %RX 0 retrans 0.08 msRTT
		[root@master ~]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.96 sec = 21707.4017 Mbps 99 %TX 85 %RX 0 retrans 0.07 msRTT
		[root@master ~]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.91 sec = 21973.6805 Mbps 100 %TX 87 %RX 0 retrans 0.08 msRTT
		[root@master ~]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.82 sec = 22483.9767 Mbps 99 %TX 87 %RX 0 retrans 0.08 msRTT
		[root@master ~]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.88 sec = 22145.0598 Mbps 99 %TX 87 %RX 0 retrans 0.08 msRTT

- Native模式下：容器安装Nuttcp client，物理主机安装Nuttcp server（测试脚本2）  


	启动参数

		docker run -it --rm --net=host centos:7.1.1503 /bin/bash

	执行结果 

		[root@master /]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.93 sec = 21876.1464 Mbps 99 %TX 86 %RX 0 retrans 0.08 msRTT
		[root@master /]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.79 sec = 22681.9264 Mbps 100 %TX 88 %RX 0 retrans 0.08 msRTT
		[root@master /]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.76 sec = 22831.7112 Mbps 99 %TX 88 %RX 0 retrans 0.08 msRTT
		[root@master /]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.88 sec = 22156.9124 Mbps 99 %TX 88 %RX 0 retrans 0.08 msRTT
		[root@master /]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.79 sec = 22685.3408 Mbps 100 %TX 88 %RX 0 retrans 0.08 msRTT

- Native模式下：两个容器分别安装Nuttcp server和Nuttcp client（测试脚本2）  


	启动参数
		
		执行以下命令创建两个容器
		docker run -it --rm --net=host centos:7.1.1503 /bin/bash

	执行结果 

		[root@master /]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.80 sec = 22608.5630 Mbps 99 %TX 88 %RX 0 retrans 0.08 msRTT
		[root@master /]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.84 sec = 22381.1590 Mbps 99 %TX 88 %RX 0 retrans 0.07 msRTT
		[root@master /]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.81 sec = 22559.3286 Mbps 99 %TX 88 %RX 0 retrans 0.08 msRTT
		[root@master /]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.78 sec = 22706.4008 Mbps 99 %TX 88 %RX 0 retrans 0.09 msRTT
		[root@master /]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /   3.75 sec = 22895.9575 Mbps 99 %TX 88 %RX 0 retrans 0.08 msRTT

# Bridge

- 容器安装iPerf server，物理主机安装iPerf client（测试脚本1）
   		
		启动参数：
		docker run -it --rm centos:7.1.1503 /bin/bash

		执行结果：
		[  4] local 172.17.42.1 port 58178 connected to 172.17.0.1 port 5201
		[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
		[  4]   0.00-1.00   sec  3.90 GBytes  33.5 Gbits/sec    0    274 KBytes       
		[  4]   1.00-2.00   sec  4.01 GBytes  34.4 Gbits/sec    0    283 KBytes       
		[  4]   2.00-3.00   sec  3.99 GBytes  34.3 Gbits/sec    0    284 KBytes       
		[  4]   3.00-4.00   sec  4.00 GBytes  34.3 Gbits/sec    0    290 KBytes       
		[  4]   4.00-5.00   sec  4.05 GBytes  34.8 Gbits/sec    0    297 KBytes       
		[  4]   5.00-6.00   sec  4.05 GBytes  34.8 Gbits/sec    0    320 KBytes       
		[  4]   6.00-7.00   sec  4.06 GBytes  34.9 Gbits/sec    0    324 KBytes       
		[  4]   7.00-8.00   sec  4.04 GBytes  34.7 Gbits/sec    0    328 KBytes       
		[  4]   8.00-9.00   sec  4.06 GBytes  34.9 Gbits/sec    0    328 KBytes       
		[  4]   9.00-10.00  sec  4.09 GBytes  35.1 Gbits/sec    0    335 KBytes       
		- - - - - - - - - - - - - - - - - - - - - - - - -
		[ ID] Interval           Transfer     Bandwidth       Retr
		[  4]   0.00-10.00  sec  40.2 GBytes  34.6 Gbits/sec    0             sender
		[  4]   0.00-10.00  sec  40.2 GBytes  34.6 Gbits/sec                  receiver

		iperf Done.

- 容器安装iPerf client，物理主机安装iPerf server（测试脚本1） 
 
	启动参数：

		docker run -it --rm centos:7.1.1503 /bin/bash

	执行结果：

		Connecting to host 133.133.134.71, port 5201
		[  4] local 172.17.0.1 port 58188 connected to 133.133.134.71 port 5201
		[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
		[  4]   0.00-1.00   sec  3.86 GBytes  33.1 Gbits/sec    0    259 KBytes       
		[  4]   1.00-2.00   sec  3.97 GBytes  34.1 Gbits/sec    0    269 KBytes       
		[  4]   2.00-3.00   sec  3.97 GBytes  34.1 Gbits/sec    0    283 KBytes       
		[  4]   3.00-4.00   sec  3.97 GBytes  34.1 Gbits/sec    0    290 KBytes       
		[  4]   4.00-5.00   sec  3.99 GBytes  34.3 Gbits/sec    0    297 KBytes       
		[  4]   5.00-6.00   sec  4.03 GBytes  34.6 Gbits/sec    0    310 KBytes       
		[  4]   6.00-7.00   sec  4.06 GBytes  34.9 Gbits/sec    0    322 KBytes       
		[  4]   7.00-8.00   sec  4.05 GBytes  34.8 Gbits/sec    0    322 KBytes       
		[  4]   8.00-9.00   sec  4.02 GBytes  34.5 Gbits/sec    0    322 KBytes       
		[  4]   9.00-10.00  sec  3.97 GBytes  34.1 Gbits/sec    0    324 KBytes       
		- - - - - - - - - - - - - - - - - - - - - - - - -
		[ ID] Interval           Transfer     Bandwidth       Retr
		[  4]   0.00-10.00  sec  39.9 GBytes  34.3 Gbits/sec    0             sender
		[  4]   0.00-10.00  sec  39.9 GBytes  34.3 Gbits/sec                  receiver
		
		iperf Done.

- 两个容器分别安装iPerf server和iPerf client（测试脚本1）

	启动参数：

		用以下命令启动两台容器：
		docker run -it --rm centos:7.1.1503 /bin/bash

    执行结果：

		Connecting to host 172.17.0.1, port 5201
		[  4] local 172.17.0.2 port 51871 connected to 172.17.0.1 port 5201
		[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
		[  4]   0.00-1.00   sec  3.77 GBytes  32.4 Gbits/sec    0    274 KBytes       
		[  4]   1.00-2.00   sec  3.94 GBytes  33.8 Gbits/sec    0    277 KBytes       
		[  4]   2.00-3.00   sec  3.95 GBytes  33.9 Gbits/sec    0    301 KBytes       
		[  4]   3.00-4.00   sec  3.91 GBytes  33.6 Gbits/sec    0    318 KBytes       
		[  4]   4.00-5.00   sec  3.93 GBytes  33.8 Gbits/sec    0    322 KBytes       
		[  4]   5.00-6.00   sec  3.93 GBytes  33.7 Gbits/sec    0    322 KBytes       
		[  4]   6.00-7.00   sec  3.94 GBytes  33.9 Gbits/sec    0    322 KBytes       
		[  4]   7.00-8.00   sec  3.93 GBytes  33.7 Gbits/sec    0    322 KBytes       
		[  4]   8.00-9.00   sec  3.94 GBytes  33.8 Gbits/sec    0    322 KBytes       
		[  4]   9.00-10.00  sec  3.94 GBytes  33.9 Gbits/sec    0    322 KBytes       
		- - - - - - - - - - - - - - - - - - - - - - - - -
		[ ID] Interval           Transfer     Bandwidth       Retr
		[  4]   0.00-10.00  sec  39.2 GBytes  33.6 Gbits/sec    0             sender
		[  4]   0.00-10.00  sec  39.2 GBytes  33.6 Gbits/sec                  receiver
		
		iperf Done.


- 容器安装Nuttcp server，物理主机安装Nuttcp client（测试脚本2）

	启动参数：

		docker run -it --rm centos:7.1.1503 /bin/bash

    执行结果：

		[root@master ~]# nuttcp -n10G -l1500 172.17.0.1  
		10239.9988 MB /  46.20 sec = 1859.3280 Mbps 100 %TX 90 %RX 0 retrans 0.10 msRTT  
		[root@master ~]# nuttcp -n10G -l1500 172.17.0.1  
		10239.9988 MB /  45.90 sec = 1871.4953 Mbps 100 %TX 90 %RX 0 retrans 0.09 msRTT  
		[root@master ~]# nuttcp -n10G -l1500 172.17.0.1  
		10239.9988 MB /  46.46 sec = 1849.0462 Mbps 100 %TX 90 %RX 0 retrans 0.09 msRTT  
		[root@master ~]# nuttcp -n10G -l1500 172.17.0.1  
		10239.9988 MB /  46.16 sec = 1860.7222 Mbps 100 %TX 90 %RX 0 retrans 0.10 msRTT  
		[root@master ~]# nuttcp -n10G -l1500 172.17.0.1  
		10239.9988 MB /  45.77 sec = 1876.9240 Mbps 100 %TX 90 %RX 0 retrans 0.10 msRTT  

		

- 容器安装Nuttcp client，物理主机安装Nuttcp server（测试脚本2）

	启动参数：

		docker run -it --rm centos:7.1.1503 /bin/bash

    执行结果：

		[root@59c4fac6fc84 /]# nuttcp -n10G -l1500 133.133.134.71  
		10239.9988 MB /  42.70 sec = 2011.6665 Mbps 100 %TX 92 %RX 0 retrans 0.09 msRTT    
		[root@59c4fac6fc84 /]# nuttcp -n10G -l1500 133.133.134.71  
		10239.9988 MB /  42.38 sec = 2027.0552 Mbps 100 %TX 92 %RX 0 retrans 0.11 msRTT    
		[root@59c4fac6fc84 /]# nuttcp -n10G -l1500 133.133.134.71  
		10239.9988 MB /  42.30 sec = 2030.7767 Mbps 100 %TX 92 %RX 0 retrans 0.09 msRTT  
		[root@59c4fac6fc84 /]# nuttcp -n10G -l1500 133.133.134.71  
		10239.9988 MB /  42.52 sec = 2020.1468 Mbps 100 %TX 91 %RX 0 retrans 0.09 msRTT  
		[root@59c4fac6fc84 /]# nuttcp -n10G -l1500 133.133.134.71  
		10239.9988 MB /  42.87 sec = 2003.8831 Mbps 100 %TX 92 %RX 0 retrans 0.09 msRTT  


- 两个容器分别安装Nuttcp server和Nuttcp client（测试脚本2）

	启动参数：

		用以下命令启动两个容器：
		docker run -it --rm centos:7.1.1503 /bin/bash

    执行结果：

		[root@59c4fac6fc84 /]# nuttcp -n10G -l1500 172.17.0.1
		10239.9988 MB /  48.86 sec = 1758.1473 Mbps 100 %TX 90 %RX 0 retrans 0.09 msRTT
		[root@59c4fac6fc84 /]# nuttcp -n10G -l1500 172.17.0.1
		10239.9988 MB /  48.83 sec = 1759.3251 Mbps 100 %TX 88 %RX 0 retrans 0.08 msRTT
		[root@59c4fac6fc84 /]# nuttcp -n10G -l1500 172.17.0.1
		10239.9988 MB /  49.25 sec = 1744.3209 Mbps 100 %TX 90 %RX 0 retrans 0.09 msRTT
		[root@59c4fac6fc84 /]# nuttcp -n10G -l1500 172.17.0.1
		10239.9988 MB /  48.80 sec = 1760.1777 Mbps 100 %TX 89 %RX 0 retrans 0.10 msRTT
		[root@59c4fac6fc84 /]# nuttcp -n10G -l1500 172.17.0.1
		10239.9988 MB /  48.90 sec = 1756.4733 Mbps 100 %TX 88 %RX 0 retrans 0.10 msRTT

# OVS  

##配置ovs

1. 安装 OpenvSwitch
2. 下载pipework

		git clone git clone https://github.com/jpetazzo/pipework.git

3. 创建两个容器用于测试, 网络模式指定为none

		docker run -it --rm --net=none --privileged=true --name test1 centos:7.1.1503 /bin/bash
		docker run -it --rm --net=none --privileged=true --name test2 centos:7.1.1503 /bin/bash

4. 创建一个网桥

		ovs-vsctl add-br ovstest
		ip linke set ovstest up
		ip addr add 133.133.134.71/16

5. 用pipework为容器创建网卡并桥接到网桥

		pipework ovstest test1 133.133.134.100/16@133.133.134.71
		pipework ovstest test2 133.133.134.101/16@133.133.134.71

##执行测试

- Ovs模式下：容器安装iPerf server，物理主机安装iPerf client（测试脚本1）  

	测试结果

		Connecting to host 133.133.134.100, port 5201
		[  4] local 133.133.134.71 port 55260 connected to 133.133.134.100 port 5201
		[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
		[  4]   0.00-1.00   sec  3.75 GBytes  32.2 Gbits/sec    0    287 KBytes       
		[  4]   1.00-2.00   sec  3.85 GBytes  33.1 Gbits/sec    4    259 KBytes       
		[  4]   2.00-3.00   sec  3.90 GBytes  33.5 Gbits/sec    0    259 KBytes       
		[  4]   3.00-4.00   sec  3.90 GBytes  33.5 Gbits/sec    0    273 KBytes       
		[  4]   4.00-5.00   sec  3.91 GBytes  33.6 Gbits/sec    0    257 KBytes       
		[  4]   5.00-6.00   sec  3.88 GBytes  33.4 Gbits/sec    0    270 KBytes       
		[  4]   6.00-7.00   sec  3.90 GBytes  33.5 Gbits/sec    0    270 KBytes       
		[  4]   7.00-8.00   sec  3.90 GBytes  33.5 Gbits/sec    0    260 KBytes       
		[  4]   8.00-9.00   sec  3.86 GBytes  33.2 Gbits/sec    0    266 KBytes       
		[  4]   9.00-10.00  sec  3.90 GBytes  33.5 Gbits/sec    0    280 KBytes       
		- - - - - - - - - - - - - - - - - - - - - - - - -
		[ ID] Interval           Transfer     Bandwidth       Retr
		[  4]   0.00-10.00  sec  38.8 GBytes  33.3 Gbits/sec    4             sender
		[  4]   0.00-10.00  sec  38.8 GBytes  33.3 Gbits/sec                  receiver
		
		iperf Done.


- Ovs模式下：容器安装iPerf client，物理主机安装iPerf server（测试脚本1）  

	测试结果

		Connecting to host 133.133.134.71, port 5201
		[  4] local 133.133.134.100 port 43336 connected to 133.133.134.71 port 5201
		[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
		[  4]   0.00-1.00   sec  3.75 GBytes  32.2 Gbits/sec    0    283 KBytes       
		[  4]   1.00-2.00   sec  3.87 GBytes  33.3 Gbits/sec    0    298 KBytes       
		[  4]   2.00-3.00   sec  3.89 GBytes  33.4 Gbits/sec    0    321 KBytes       
		[  4]   3.00-4.00   sec  3.89 GBytes  33.4 Gbits/sec    0    321 KBytes       
		[  4]   4.00-5.00   sec  3.93 GBytes  33.8 Gbits/sec    0    324 KBytes       
		[  4]   5.00-6.00   sec  3.93 GBytes  33.7 Gbits/sec    0    325 KBytes       
		[  4]   6.00-7.00   sec  3.91 GBytes  33.6 Gbits/sec    0    327 KBytes       
		[  4]   7.00-8.00   sec  3.91 GBytes  33.6 Gbits/sec    0    327 KBytes       
		[  4]   8.00-9.00   sec  3.89 GBytes  33.4 Gbits/sec    0    327 KBytes       
		[  4]   9.00-10.00  sec  3.90 GBytes  33.5 Gbits/sec    0    327 KBytes       
		- - - - - - - - - - - - - - - - - - - - - - - - -
		[ ID] Interval           Transfer     Bandwidth       Retr
		[  4]   0.00-10.00  sec  38.9 GBytes  33.4 Gbits/sec    0             sender
		[  4]   0.00-10.00  sec  38.9 GBytes  33.4 Gbits/sec                  receiver
		
		iperf Done.


- Ovs模式下：两个容器分别安装iPerf server和iPerf client（测试脚本1）  

	执行结果：

		Connecting to host 133.133.134.209, port 5201
		[  4] local 133.133.134.211 port 33766 connected to 133.133.134.209 port 5201
		[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
		[  4]   0.00-1.00   sec  3.76 GBytes  32.3 Gbits/sec    0    263 KBytes       
		[  4]   1.00-2.00   sec  3.88 GBytes  33.3 Gbits/sec    0    287 KBytes       
		[  4]   2.00-3.00   sec  3.84 GBytes  33.0 Gbits/sec    0    308 KBytes       
		[  4]   3.00-4.00   sec  3.86 GBytes  33.1 Gbits/sec    0    311 KBytes       
		[  4]   4.00-5.00   sec  3.90 GBytes  33.5 Gbits/sec    0    322 KBytes       
		[  4]   5.00-6.00   sec  3.86 GBytes  33.2 Gbits/sec    0    324 KBytes       
		[  4]   6.00-7.00   sec  3.87 GBytes  33.3 Gbits/sec    0    327 KBytes       
		[  4]   7.00-8.00   sec  3.88 GBytes  33.3 Gbits/sec    0    327 KBytes       
		[  4]   8.00-9.00   sec  3.87 GBytes  33.3 Gbits/sec    0    328 KBytes       
		[  4]   9.00-10.00  sec  3.87 GBytes  33.2 Gbits/sec    0    331 KBytes       
		- - - - - - - - - - - - - - - - - - - - - - - - -
		[ ID] Interval           Transfer     Bandwidth       Retr
		[  4]   0.00-10.00  sec  38.6 GBytes  33.1 Gbits/sec    0             sender
		[  4]   0.00-10.00  sec  38.6 GBytes  33.1 Gbits/sec                  receiver
		
		iperf Done.


- Ovs模式下：容器安装Nuttcp server，物理主机安装Nuttcp client（测试脚本2）
  
	执行结果：

		[root@master test-tools]# nuttcp -n10G -l1500 133.133.134.100
		10239.9988 MB /  49.55 sec = 1733.5549 Mbps 100 %TX 90 %RX 0 retrans 0.10 msRTT
		[root@master test-tools]# nuttcp -n10G -l1500 133.133.134.100
		10239.9988 MB /  51.54 sec = 1666.7745 Mbps 100 %TX 89 %RX 0 retrans 0.10 msRTT
		[root@master test-tools]# nuttcp -n10G -l1500 133.133.134.100
		10239.9988 MB /  50.19 sec = 1711.5304 Mbps 100 %TX 90 %RX 0 retrans 0.10 msRTT
		[root@master test-tools]# nuttcp -n10G -l1500 133.133.134.100
		10239.9988 MB /  50.56 sec = 1699.1015 Mbps 100 %TX 90 %RX 0 retrans 0.11 msRTT
		[root@master test-tools]# nuttcp -n10G -l1500 133.133.134.100
		10239.9988 MB /  51.50 sec = 1667.9682 Mbps 100 %TX 90 %RX 0 retrans 0.09 msRTT
		[root@master test-tools]# nuttcp -n10G -l1500 133.133.134.100

- Ovs模式下：容器安装Nuttcp client，物理主机安装Nuttcp server（测试脚本2）  

	执行结果

		[root@176c0b44d6bf tmp]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /  44.48 sec = 1931.1249 Mbps 100 %TX 92 %RX 0 retrans 0.09 msRTT
		[root@176c0b44d6bf tmp]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /  44.04 sec = 1950.3362 Mbps 100 %TX 91 %RX 0 retrans 0.10 msRTT
		[root@176c0b44d6bf tmp]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /  44.07 sec = 1949.0209 Mbps 100 %TX 91 %RX 0 retrans 0.09 msRTT
		[root@176c0b44d6bf tmp]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /  44.11 sec = 1947.4205 Mbps 100 %TX 92 %RX 0 retrans 0.09 msRTT
		[root@176c0b44d6bf tmp]# nuttcp -n10G -l1500 133.133.134.71
		10239.9988 MB /  43.97 sec = 1953.6540 Mbps 100 %TX 91 %RX 0 retrans 0.09 msRTT


- Ovs模式下：两个容器分别安装Nuttcp server和Nuttcp client（测试脚本2）

	执行结果：  

		[root@cca7790c3f8e tmp]# nuttcp -n10G -l1500 133.133.134.209
		10239.9988 MB /  50.17 sec = 1712.1890 Mbps 100 %TX 90 %RX 0 retrans 0.10 msRTT
		[root@cca7790c3f8e tmp]# nuttcp -n10G -l1500 133.133.134.209
		10239.9988 MB /  49.21 sec = 1745.6593 Mbps 100 %TX 90 %RX 0 retrans 0.10 msRTT
		[root@cca7790c3f8e tmp]# nuttcp -n10G -l1500 133.133.134.209
		10239.9988 MB /  50.37 sec = 1705.3083 Mbps 100 %TX 90 %RX 0 retrans 0.09 msRTT
		[root@cca7790c3f8e tmp]# nuttcp -n10G -l1500 133.133.134.209
		10239.9988 MB /  50.04 sec = 1716.7725 Mbps 100 %TX 90 %RX 0 retrans 0.09 msRTT
		[root@cca7790c3f8e tmp]# nuttcp -n10G -l1500 133.133.134.209
		10239.9988 MB /  49.56 sec = 1733.1538 Mbps 100 %TX 90 %RX 0 retrans 0.09 msRTT

# Conclusion  

##Comparsion of Native, Bridge, OVS Network Bandwith
 
![](http://i.imgur.com/yy3VDPb.png)

##Comparsion of Native, Bridge, OVS Network Latency

![](http://i.imgur.com/Io8XqAj.png)