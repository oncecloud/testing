#Devicemapper 实验  

Devicemapper作为Docker一个存储驱动，用于存储Docker容器和镜像。Docker daemon启动时默认大小设为100G，每个容器可用容量默认为10G，可通过--storage-opt选项在启动时设置devicemapper大小。本文主要验证，在挂载和不挂载宿主机目录的情况下，devicemapper存储资源耗尽时容器的运行情况。

## 需求

启动一个容器，设置总存储大小为10G， pull一个镜像用于实验。

	docker daemon --storage-driver=devicemapper --storage-opt dm.loopdatasize=10G
	
	docker pull ubuntu

	docker info:
	
	 Containers: 0
	 Images: 4
	 Storage Driver: devicemapper
	 Pool Name: docker-253:1-203357852-pool
	 Pool Blocksize: 65.54 kB
	 Backing Filesystem: xfs
	 Data file: /dev/loop0
	 Metadata file: /dev/loop1
	 Data Space Used: 2.057 GB
	 Data Space Total: 10.74 GB
	 Data Space Available: 8.681 GB
	 Metadata Space Used: 1.364 MB
	 Metadata Space Total: 2.147 GB
	 Metadata Space Available: 2.146 GB
	 Udev Sync Supported: true
	 Deferred Removal Enabled: false
	 Data loop file: /var/lib/docker/devicemapper/devicemapper/data
	 Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
	 Library Version: 1.02.93-RHEL7 (2015-01-28)
	 Execution Driver: native-0.2
	 Logging Driver: json-file
	 Kernel Version: 3.10.0-229.14.1.el7.x86_64
	 Operating System: CentOS Linux 7 (Core)
	 CPUs: 2
	 Total Memory: 1.784 GiB
	 Name: localhost.localdomain
	 ID: APCF:P2ON:RANA:H3P3:TGHF:OBPC:2E3Y:JEDA:6PA4:GCJN:A74B:DRIN
	 WARNING: bridge-nf-call-iptables is disabled
	 WARNING: bridge-nf-call-ip6tables is disabled 

##实验一 启动容器， 挂载外部目录

	docker run -it -v /tmp/data:/tmp ubuntu /bin/bash

用dd写入12G大小文件：

	dd if=/dev/zero of=/tmp/testfile bs=4M count=2816 conv=fdatasync

	2816+0 records in
	2816+0 records out
	11811160064 bytes (12 GB) copied, 87.9712 s, 134 MB/s  

文件成功写入宿主机目录。

	docker info:
		
	Containers: 1
	Images: 4
	Storage Driver: devicemapper
	 Pool Name: docker-253:1-203357852-pool
	 Pool Blocksize: 65.54 kB
	 Backing Filesystem: xfs
	 Data file: /dev/loop0
	 Metadata file: /dev/loop1
	 Data Space Used: 2.059 GB
	 Data Space Total: 10.74 GB
	 Data Space Available: 8.679 GB
	 Metadata Space Used: 1.397 MB
	 Metadata Space Total: 2.147 GB
	 Metadata Space Available: 2.146 GB
	 Udev Sync Supported: true
	 Deferred Removal Enabled: false
	 Data loop file: /var/lib/docker/devicemapper/devicemapper/data
	 Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
	 Library Version: 1.02.93-RHEL7 (2015-01-28)
	Execution Driver: native-0.2
	Logging Driver: json-file
	Kernel Version: 3.10.0-229.14.1.el7.x86_64
	Operating System: CentOS Linux 7 (Core)
	CPUs: 2
	Total Memory: 1.784 GiB
	Name: localhost.localdomain
	ID: APCF:P2ON:RANA:H3P3:TGHF:OBPC:2E3Y:JEDA:6PA4:GCJN:A74B:DRIN
	WARNING: bridge-nf-call-iptables is disabled
	WARNING: bridge-nf-call-ip6tables is disabled

##实验二 启动容器， 不挂载外部目录 

	docker run -it ubutu /bin/bash  

用dd写入12G大小文件： 

	dd if=/dev/zero of=/tmp/testfile bs=4M count=2816 conv=fdatasync

	dd: error writing '/tmp/testfile': Read-only file system
	2132+0 records in
	2131+0 records out
	8939671552 bytes (8.9 GB) copied, 134.91 s, 66.3 MB/s 

只写入8.9G,且storage pool写满，文件系统设为只读模式。 

	docker info:
	
	Containers: 1
	Images: 4
	Storage Driver: devicemapper
	 Pool Name: docker-253:1-203357852-pool
	 Pool Blocksize: 65.54 kB
	 Backing Filesystem: xfs
	 Data file: /dev/loop0
	 Metadata file: /dev/loop1
	 Data Space Used: 10.74 GB
	 Data Space Total: 10.74 GB
	 Data Space Available: 0 B
	 Metadata Space Used: 5.779 MB
	 Metadata Space Total: 2.147 GB
	 Metadata Space Available: 2.142 GB
	 Udev Sync Supported: true
	 Deferred Removal Enabled: false
	 Data loop file: /var/lib/docker/devicemapper/devicemapper/data
	 Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
	 Library Version: 1.02.93-RHEL7 (2015-01-28)
	Execution Driver: native-0.2
	Logging Driver: json-file
	Kernel Version: 3.10.0-229.14.1.el7.x86_64
	Operating System: CentOS Linux 7 (Core)
	CPUs: 2
	Total Memory: 1.784 GiB
	Name: localhost.localdomain
	ID: APCF:P2ON:RANA:H3P3:TGHF:OBPC:2E3Y:JEDA:6PA4:GCJN:A74B:DRIN
	WARNING: bridge-nf-call-iptables is disabled
	WARNING: bridge-nf-call-ip6tables is disabled

##结论  
1. Devicemapper存储池用来存储容器和镜像，默认大小100G，每个容器最大10G。
2. 当分配的存储池资源用完时，devicemapper设为只读模式，需要备份容器和镜像，进行扩       容。
3. 容器挂载外部目录并把文件写入外部目录时，文件存储在宿主机目录，不占用devicemapper资源。

