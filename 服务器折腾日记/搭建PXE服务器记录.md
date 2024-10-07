# 搭建PXE服务器记录

环境Ubuntu Server22.04，Raspberry Pi 3B。

目标平台amd64

## 1、PXE启动流程

略，看OS笔记。



## 2、必备软件以及功能

1. PXE-enable DHCP server：这里选择的是isc-dhcp-server（也称[DHCPD](https://en.wikipedia.org/wiki/DHCPD)），一种DHCP server最早也最知名的实现。还有其它许多的DHCP server的实现，比如[dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq)，[Kea DHCP](https://en.wikipedia.org/wiki/Kea_(software))等等。

2. TFTP server：tftpd-hpa。

3. apache2：Apache HTTP server。用以传输完整的os镜像文件。

4. vsftpd：ftp server，与apache2目的相同。

5. nfs-kernel-server：NFS服务器，与上面二者目的相同，三选一即可(一般名字最后带d的是服务器daemon，不带的是客户端)。

6. **[Syslinux](https://wiki.archlinux.org/title/syslinux)**：是能够从drives，CDs以及PXE启动的一个boot loader的集合。

   包括5个boot loaders：

   - SYSLINUX：用于从FAT filesystem启动
   - ISOLINUX：用于从ISO 9660 filesystem启动
   - PXELINUX：用于从PXE启动
   - EXTLINUX： 用于从[Btrfs](https://en.wikipedia.org/wiki/Btrfs)，[ext2](https://en.wikipedia.org/wiki/Ext2)，[ext3](https://en.wikipedia.org/wiki/Ext3)，[ext4](https://en.wikipedia.org/wiki/Ext4)，[FAT](https://en.wikipedia.org/wiki/File_Allocation_Table)，[NTFS](https://en.wikipedia.org/wiki/NTFS)，[UFS/UFS2](https://en.wikipedia.org/wiki/Unix_File_System)和[XFS](https://en.wikipedia.org/wiki/XFS) filesystems启动
   - MEMDISK：为旧的OS（像MS-DOS）模拟RAM disk

   搭建PXE服务器过程中用到了syslinux与pxelinux，用以PXE以Legacy BIOS方式启动。

   安装好syslinux与pxelinux后需要将/usr/lib/syslinux/modules/bios/中的ldlinux.c32，libutil.c32，menu.c32，vesamenu.c32等文件拷贝一份到相应tftp服务根目录的Legacy下，同时将/usr/lib/PXELINUX/下的lpxelinux.0,pxelinux.0也拷贝一份。

   **原因**：

7. shim-signed：Secure Boot chain-loading bootloader (Microsoft-signed binary)。shim-signed提供了一个最小的boot loader。

8. grub-efi-amd64-signed：

9. grub-common：包括GRUB Legacy和GRUB 2的一些common files。

10. kickstart

其他可选的软件：

1. xinetd：是一个daemon用于管理网络连接，用以取代inetd
2. dnsmasq：同时实现了 DHCP、TFTP、DNS 三种服务器
3. Nginx：另一种HTTP服务器
4. [radvd](https://en.wikipedia.org/wiki/Radvd)：
5. net-tools：包含了一系列有用的网络工具比如netstat



## 3、相关配置以及原理

### 3.1 /srv与/var

`/srv`文件夹：/srv文件夹用以SAU存储本机提供的服务与数据，如 FTP，WWW或者CVS。本机安装的TFTP server与FPT server的服务数据的根目录就是/srv/tftp和/srv/ftp。而apche2的服务数据根目录为/var/www/html。



### 3.2 TFTP server配置

TFTP server的配置文件位于/etc/default/tftpd-hpa，内部选项包括

```
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/srv/tftp"
TFTP_ADDRESS="192.168.31.172:69"
TFTP_OPTIONS="--secure"
```



### 3.3 DHCP server配置

DHCP server配置文件有两个分别位于/etc/default/osc-dhcp-server和/etc/dhcp/dhcpd.conf。

前者是脚本：/etc/init.d/isc-dhcp-server 所使用的配置文件，用于：

- 指明进程 dhcpd 配置文件的路径（ipv4 / ipv6）；
- 指明进程 dhcpd 的 PID 文件路径（ipv4 / ipv6）；
- 启动 dhcpd 时额外的选项；
- dhcpd 服务绑定的网卡的名称（ipv4 / ipv6）；

后者是进程：dhcpd 的配置文件，用以配置IP池等。



### 3.4 启动项配置

#### 3.4.1 Legacy PXE启动项

需要用到的几个文件的作用：

1. **ldlinux.c32**：
2. **libutil.c32**：
3. **menu.c32**：
4. **vesamenu.c32**：
5. **lpxelinux.0**：
6. **pxelinux.0**：

#### 3.4.2 UEFI PXE启动项
