---
title: macosX挂载NFS
date: 2019-09-30 20:09:14
tags: nfs
---

1. Mac OS X使用automounter，也称为 autofs 来挂载NFS输出卷。Autofs包含以下程序和daemons:

   * **autofsd**

     autofsd 执行 automount 之后 ，就会等待网络配置修改事件以及类似的事件发生。如果发生这样的事件，重新运行一次 automount 来更新挂载以反映当前automounter映射。也可以使用automount_reread 来运行 automount 。

   * **automountd**

     automountd 是一个响应从 autofs 发出的请求的服务，用来挂载或卸载网络文件系统，并且提供目录的内容，基于automounter映射的内容。这个 automountd 是通过 launchd 来启动的。

   * **automount**

     automount 是实际的挂载管理器。使用一些映射文件和配置文件来管理挂载和卸载远程资源。这些配置文件使用 /etc/autofs.conf 和 /etc/auto_master。

   * **automount_reread**

     automount_reread 可以触发针对 autofs 的网络变更事件。
     
     <!--more-->

2. 检查autofs相关服务

   * `ps -ef | grep auto | grep -v grep`

     > ​    0    95     1   0 六12上午 ??         0:00.03 autofsd

     可以看到系统运行了一个autofsd

   * 检查一个服务是否通过 launchd 启动

     `sudo launchctl list | grep -E 'automo|autof'`

     > 95	0	com.apple.autofsd
     > 13290	0	com.apple.automountd

3. Autofs映射，autofs 有两种映射方式

   * **Direct Map** 

     直接映射是直接列出目录的文件系统位置，关键字是完整的目录名字，例如

     ```shell
     /usr/local      eng4:/export/local
     /src            eng4:/export/src
     ```

   * **Indirect Map**

     非直接映射是为了用于将大量的对象和一个单一目录关联的情况。每个映射入口是一个目录入口的简化名字。一个非常好的案例是 auto_home 映射，可以检测所有在 /home 目录下的入口,例如：

     ```shell
     bill    argon:/export/home/bill
     brent   depot:/export/home/brent
     guy     depot:/export/home/guy
     ```
     

4. 创建AutoFS的Indirect Map

   * 先在Windows主机（win7）上设置共享目录Mac。

   * 编辑 /etc/auto_master配置文件添加如下：

     ```shell
     #
     # Automounter master map
     #
     +auto_master		# Use directory service
     /net			-hosts		-nobrowse,hidefromfinder,nosuid
     /home			auto_home	-nobrowse,hidefromfinder
     /Network/Servers	-fstab
     /-			-static
     /Users/haominglfs/win7      autofs_win7
     ```

     以上配置告知 OS X 任何位于 `/Users/haominglfs/win7` 目录下的入口都通过 `/etc/autofs_win7`配置文件来配置。

     >在auto_master前面的”+”符号表示让OS X查看目录服务（例如Open Directory，LDAP等等）是否有自动挂载记录，如果从目录服务找到自动挂载记录就使用其进行挂载。

     >/home目录被设置成 auto_home ，但是这个并不是一个完全目录，而是指 /etc/auto_home。这是一个非直接映射(indirect map)的例子。定义本地目录的挂载点，而远程挂载则是 /etc/auto_home 映射文件定义。网络用户登录将使用 /etc/auto_home 中定义的目录挂载到 /home 上。

     > 同样在 /etc/auto_home 配置文件中也可以看到 +auto_home 的配置表示查找使用目录服务的auto_home记录。

   * 编辑 `/etc/autofs_win7`内容如下

     ` mac -fstype=nfs 192.168.1.242:/e/mac`
     
   * 执行命令`showmount -e 192.168.1.242`  查看可挂载的NFS目录。

   * 执行 automount 命令挂载

     `sudo automount -vc`

​     

​     

​     

   