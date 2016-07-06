# Devicemapper空间无法释放

## Problem

在容器内删除一个文件，不能释放占用空间（从宿主机删除容器和镜像空间能够正常回收）。


## 初步分析

问题的根本原因在于在**Devicemapper**的逻辑层删除数据时，底层的块设备空间不能够正常回收。  
根据[issue3182](https://github.com/docker/docker/issues/3182)，该问题是Devicemapper内核的[bug](https://bugzilla.redhat.com/show_bug.cgi?id=1043527). 目前正在修复。

## 解决方法

该问题已经提交到社区，等待进一步追踪

目前可行的**Workaround**:

1. 使用OverlayFs驱动: Docker官方推荐使用[OverlayFs](https://github.com/docker/docker/pull/12354),有待进一步验证
2. 规避该问题：Docker最佳实践提倡把经常读写的大文件挂载到宿主机目录。


下一步对OverlayFs进行调研，并继续分析Devicemapper的一些可行方案，以尽早解决该问题适用于生产环境中。
