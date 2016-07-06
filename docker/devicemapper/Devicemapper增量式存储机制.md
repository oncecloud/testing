# Devicemapper增量式存储机制

## 摘要  

镜像增量式存储作为Docker的关键技术，是其流行的重要原因。作为Docker首选的AUFS目前尚未被编译进Linux的内核，相对于AUFS， DeviceMapper具有更好的兼容性。本文以Devicemapper为例，分析Docker增量式存储的底层原理。  
  
首先，介绍本文的研究背景，即Docker的分层文件系统， 然后介绍了Devicemapper的基本原理以及有助于理解本文的相关技术，紧接着，结合Devicemapper分析Docked增量式存储原理，Docker是如何利用Devicemapper实现分层机制。最后，本文对相关部分的源码进行了分析，验证本文分析的正确性。

----------


## 背景
典型的Linux文件系统由bootfs和rootfs两部分组成，bootfs主要包含 bootloader和kernel，bootloader主要是引导加载kernel，当kernel被加载到内存中后bootfs就被umount了。 rootfs包含的就是典型 Linux 系统中的/dev，/proc，/bin，/etc等标准目录和文件。  Linux加载bootfs时会先将rootfs设为read-only，然后在系统自检之后将rootfs从read-only改为read-write，然后我们就可以在rootfs上进行写和读的操作了。但Docker的镜像却不是这样，它在bootfs自检完毕之后并不会把rootfs的read-only改为read-write。而是利用union mount（UnionFS的一种挂载机制）将一个或多个read-only的rootfs加载到之前的read-only的rootfs层之上。在加载了这么多层的rootfs之后，仍然让它看起来只像是一个文件系统，在Docker的体系里把union mount的这些read-only的rootfs叫做Docker的镜像。但是，此时的每一层rootfs都是read-only的，我们此时还不能对其进行操作。当我们创建一个容器，也就是将Docker镜像进行实例化，系统会在一层或是多层read-only的rootfs之上分配一层空的read-write的rootfs。  
![](http://i.imgur.com/QSKnWqi.png)

----------

## Devicemapper原理及相关技术
### Devicemapper
Device Mapper 是 Linux2.6 内核中支持逻辑卷管理的通用设备映射机制，它为实现用于存储资源管理的块设备驱动提供了一个高度模块化的内核架构，它包含三个重要的对象概念，Mapped Device、Mapping Table、Target device。

Mapped Device 是一个逻辑抽象，可以理解成为内核向外提供的逻辑设备，它通过Mapping Table描述的映射关系和 Target Device 建立映射。Target device 表示的是 Mapped Device 所映射的物理空间段，对 Mapped Device 所表示的逻辑设备来说，就是该逻辑设备映射到的一个物理设备。

Mapping Table里有 Mapped Device 逻辑的起始地址、范围、和表示在 Target Device 所在物理设备的地址偏移量以及Target 类型等信息
（注：这些地址和偏移量都是以磁盘的扇区为单位的，即 512 个字节大小，所以，当你看到128的时候，其实表示的是128*512=64K）。

DeviceMapper 中的逻辑设备Mapped Device不但可以映射一个或多个物理设备Target Device，还可以映射另一个Mapped Device，于是，就是构成了一个迭代或递归的情况，就像文件系统中的目录里除了文件还可以有目录，理论上可以无限嵌套下去。

DeviceMapper在内核中通过一个一个模块化的 Target Driver 插件实现对 IO 请求的过滤或者重新定向等工作，当前已经实现的插件包括软 Raid、加密、多路径、镜像、快照等，这体现了在 Linux 内核设计中策略和机制分离的原则。如下图所示。从图中，我们可以看到DeviceMapper只是一个框架，在这个框架上，我们可以插入各种各样的策略（让我不自然地想到了面向对象中的策略模式），在这诸多“插件”中，有一个东西叫Thin Provisioning Snapshot，这是Docker使用DeviceMapper中最重要的模块。

![](http://i.imgur.com/dzxQXyv.gif )

###Docker存储结构

![](http://i.imgur.com/zYs1NI7.png)    

Docker在初始化过程中，会创建一个100G的用于存储数据和一个2G的用于存储元数据的稀疏文件，分别附加到回环块设备/dev/loop0和/dev/loop1，然后基于回环块设备创建thin pool。 Devicemapper在thin pool之上创建Thinly-Provisioned Snapshot，将多个快照挂在同一个卷下从而实现文件系统的分层。

- Loop device  
    Loop设备是一种伪设备(pseudo-device)，这种设备使得文件可以如同块设备一般被访问。  在使用之前，一个 loop设备必须要和一个文件进行连接。这种结合方式给用户提供了一个替代块特殊文件的接口。因此，如果这个文件包含有一个完整的文件系统，那么这个文件就可以像一个磁盘设备一样被mount 起来。  

- Thin provision  
    thin provision在centos6.3中作为技术预览引入,在centos6.5和centos7中全⾯⽀持。
    类似于虚拟内存技术，Thin provision是一种存储虚拟化技术，支持分配给用户超出物理内存限制的资源，但只有写入时才会占用磁盘空间。例如，某用户向服务器请求10GB的资源，但实际仅需要2GB，系统管理员准备2GB的物理存储，并给服务器分配10GB的虚拟卷。服务器即可基于仅占虚拟卷容量1/5的现有物理磁盘池开始运行。与之相对的是fat provision.

-  Thin pool  
    一个thin逻辑卷创建前必须创建thinpoolLV，thinpoolLV由两部分组成：dataLV,用来储存数据块;metadateLV,记录了thin卷中每个块数据的所属关系。（说简单点就是metadata中储存索引，data中储存真实数据，当你访问数据时，先通过索引再访问数据)  

- Snapshot  
    Snapshot是Lvm提供的一种特性，它可以在不中断服务运行的情况下为the origin（original device）创建一个虚拟快照(Snapshot) 

- Thinly-Provisioned Snapshot 

	Thin-provisioning Snapshot结合Thin-Provisioning和Snapshot两种技术，允许多个虚拟设备同时挂载到一个数据卷以达到数据共享的目的。Thin-Provisioning Snapshot的特点如下：

	- 可以将不同的snaptshot挂载到同一个the origin上，节省了磁盘空间。
	- 当多个Snapshot挂载到了同一个the origin上，并在the origin上发生写操作时，将会触发COW操作。这样不会降低效率。
	- Thin-Provisioning Snapshot支持递归操作，即一个Snapshot可以作为另一个Snapshot的the origin，且没有深度限制。
	- 在Snapshot上可以创建一个逻辑卷，这个逻辑卷在实际写操作（COW，Snapshot写操作）发生之前是不占用磁盘空间的。  

----------

## 增量式存储原理

下面，我们演示Docker是如何用Device Mapper的Thin Provisioning Snapshot来实现增量式存储，这也即是Docker的Devicemapper驱动初始化过程。



1. 建两个文件，一个是data.img，一个是metadata.img  
  
	    $ dd if=/dev/zero of=/tmp/data.img bs=1K count=1 seek=10M  
	    1+0 records in  
	    1+0 records out  
	    1024 bytes (1.0 kB) copied, 0.000621428 s, 1.6 MB/s    
	 
	    $ dd if=/dev/zero of=/tmp/meta.data.img bs=1K count=1 seek=1G  
	    1+0 records in  
	    1+0 records out  
	    1024 bytes (1.0 kB) copied, 0.000140858 s, 7.3 MB/s  

	注意命令中seek选项，其表示为略过of选项指定的输出文件的前10G个output的bloksize的空间后再写入内容。因为bs是1个字节，所以也就是10G的尺寸，但其实在硬盘上是没有占有空间的，占有空间只有1k的内容。当向其写入内容时，才会在硬盘上为其分配空间。我们可以用ls命令看一下，实际分配了12K和4K。  


2. 创建loopback设备

	    $ losetup /dev/loop2015 /tmp/data.img  
	    $ losetup /dev/loop2016 /tmp/meta.data.img   
	    $ losetup -a  
	    /dev/loop2015: [64768]:103991768 (/tmp/data.img)  
	    /dev/loop2016: [64768]:103991765 (/tmp/meta.data.img) 

3. 创建Thin pool  

	     $ dmsetup create hchen-thin-pool \
	      --table "0 20971522 thin-pool /dev/loop2016 /dev/loop2015 \
	       128 65536 1 skip_block_zeroing"    
	
	    $ ll /dev/mapper/hchen-thin-pool
	    lrwxrwxrwx. 1 root root 7 Aug 25 23:24 /dev/mapper/hchen-thin-pool -> ../dm-4

4. 创建Thin Provisioning Volume
  
    	$ dmsetup message /dev/mapper/hchen-thin-pool 0 "create_thin 0"  
   		$ dmsetup create hchen-thin-volumn-001 \
    		-table "0 2097152 thin /dev/mapper/hchen-thin-pool 0"  
 
5. 格式化    

	    $ mkfs.ext4 /dev/mapper/hchen-thin-volumn-001  
	    mke2fs 1.42.9 (28-Dec-2013)  
	    Discarding device blocks: done  
	    Filesystem label=  
	    OS type: Linux  
	    Block size=4096 (log=2)  
	    Fragment size=4096 (log=2)  
	    Stride=16 blocks, Stripe width=16 blocks  
	    65536 inodes, 262144 blocks  
	    13107 blocks (5.00%) reserved for the super user  
	    First data block=0  
	    Maximum filesystem blocks=268435456  
	    8 block groups  
	    32768 blocks per group, 32768 fragments per group  
	    8192 inodes per group  
	    Superblock backups stored on blocks:  
	    32768, 98304, 163840, 229376  
	     
	    Allocating group tables: done  
	    Writing inode tables: done  
	    Creating journal (8192 blocks): done  
	    Writing superblocks and filesystem accounting information: done  

6. mount  

	    $ mkdir -p /mnt/base  
	    $ mount /dev/mapper/hchen-thin-volumn-001 /mnt/base  
	    $ echo "hello world, I am a base" > /mnt/base/id.txt  
	    $ cat /mnt/base/id.txt  
	    hello world, I am a base  

7. snapshot   

		$ dmsetup message /dev/mapper/hchen-thin-pool 0 "create_snap 1 0"  
    	$ dmsetup create mysnap1 \  
       		--table "0 2097152 thin /dev/mapper/hchen-thin-pool 1"    
    	$ ll /dev/mapper/mysnap1  
    	lrwxrwxrwx. 1 root root 7 Aug 25 23:49 /dev/mapper/mysnap1 -> ../dm-5    

	上面的命令中：  
	第一条命令是向hchen-thin-pool发一个create_snap的消息，后面跟两个id，第一个是新的dev id，第二个是要从哪个已有的dev id上做snapshot（0这个dev id是我们前面就创建了了）  
	第二条命令是创建一个mysnap1的device，并可以被mount。  

    	$ mkdir -p /mnt/mysnap1
    	$ mount /dev/mapper/mysnap1 /mnt/mysnap1
    	 
   		$ ll /mnt/mysnap1/
    	total 20
    	-rw-r--r--. 1 root root 25 Aug 25 23:46 id.txt
   	 	drwx------. 2 root root 16384 Aug 25 23:43 lost+found  
     
    	$ cat /mnt/mysnap1/id.txt  
    	hello world, I am a base   

    修改一下/mnt/mysnap1/id.txt，并加上一个snap1.txt的文件：  

     	$ echo "I am snap1" >> /mnt/mysnap1/id.txt  
     	$ echo "I am snap1" > /mnt/mysnap1/snap1.txt  
     
     	$ cat /mnt/mysnap1/id.txt  
      	hello world, I am a base  
      	I am snap1  
     
     	$ sudo cat /mnt/mysnap1/snap1.txt  
     	 I am snap1
    
    再看一下/mnt/base，你会发现没有什么变化：  
  
    	$ ls /mnt/base  
    	id.txt  lost+found  
    	$ cat /mnt/base/id.txt  
    	hello world, I am a base   
 
    你是不是已经看到了分层镜像的样子了？  


8. 递归snapshot  
    
    在刚才的snapshot上再建一个snapshot      

    	$ sudo dmsetup message /dev/mapper/hchen-thin-pool 0 "create_snap 2 1"  
    	$ sudo dmsetup create mysnap2 \  
       		--table "0 2097152 thin /dev/mapper/hchen-thin-pool 2"  
     
    	$ sudo ll /dev/mapper/mysnap2  
    	lrwxrwxrwx. 1 root root 7 Aug 25 23:52 /dev/mapper/mysnap1 -> ../dm-7  
     
    	$ sudo mkdir -p /mnt/mysnap2  
    	$ sudo mount /dev/mapper/mysnap2 /mnt/mysnap2  
    	$ sudo  ls /mnt/mysnap2  
    	id.txt  lost+found  snap1.txt  
	    

至此，分层镜像Demo演示完毕。  

----------

## 源码分析
**努力构建中...**  
预计9月30日晚11点之前完成！

