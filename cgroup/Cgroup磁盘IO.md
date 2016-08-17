# Cgroup磁盘限速简单测试

## BenchMark Spec

Spec |  Matrix
				----|------|
				Environment | Dell OPTIPLEX 9010
				Software | **dd命令，iotop命令**
				CPU |  4 Core x Intel(R) Core(TM) i5-3470 CPU @ 3.20GHz
				Memory | 8G
				Nic |  100Mb/s
				Operating System | CentOS Linux 7.1.1503 (Core)
				Kernel | 3.10.0-229.4.2.el7.x86_64

## 测试用例

#### 1.直接磁盘I/O (oflag=direct)

**无限制**
```
dd if=/dev/zero of=test bs=4K count=262140 oflag=direct
```

```
262140+0 records in
262140+0 records out
1073725440 bytes (1.1 GB) copied, 19.7428 s, 54.4 MB/s

```

**Cgroup**限制为10M

```
mkdir /sys/fs/cgroup/blkio/test
echo '253:0 10485760' > /sys/fs/cgroup/blkio/test/blkio.throttle.write_bps_device
dd if=/dev/zero of=test bs=4K count=262140 oflag=direct | echo $(pidof dd) > /sys/fs/cgroup/blkio/test/tasks

```

```
262140+0 records in
262140+0 records out
1073725440 bytes (1.1 GB) copied, 102.44 s, 10.5 MB/s
```

直接I/O模式下，Cgroup能够准确限制住磁盘写速度。
但是，采用iotop产看时，发现磁盘速度会经过由50M/s下降到10M/s的短暂过程（持续500ms左右）

#### 2.缓存I/O

**无限制**

```
dd if=/dev/zero of=test bs=4K count=262140
```
```
262140+0 records in
262140+0 records out
1073725440 bytes (1.1 GB) copied, 11.9019 s, 90.2 MB/s
```

**Cgroup**限制为10M

```
mkdir /sys/fs/cgroup/blkio/test
echo '253:0 10485760' > /sys/fs/cgroup/blkio/test/blkio.throttle.write_bps_device
dd if=/dev/zero of=test bs=4K count=262140 | echo $(pidof dd) > /sys/fs/cgroup/blkio/test/tasks
```

```
262140+0 records in
262140+0 records out
1073725440 bytes (1.1 GB) copied, 11.771 s, 91.2 MB/s

```

## 实验结果

- 直接I/O模式(oflag=direct)下，Cgroup能够准确限制住磁盘I/O速度
- 缓存模式下，Cgroup不能限制住磁盘I/O速度
- 具体应用场景中，磁盘读写默认开启内核缓存，因此会出现Cgroup限制不准确现象

## 原因分析

Next...
