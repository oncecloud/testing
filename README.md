# 容器夸主机网络overlay和ovs性能测试

## 摘要
Docker1.9+提供覆盖网络(overlay)，用于支持夸主机通信。本文选取overlay网络模式和原生ovs网络模式，考虑CPU资源消耗、网络吞吐量和延迟三个度量指标，对比容器夸主机通信的性能。

## 实验设计 

#### 测试环境

- 两台ThinkCenter M8000+主机，配置信息见下表(待补充)
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
- [OVS网络安装](https://github.com/oncecloud/testing/blob/master/ovs-install.md)

## 实验结果
