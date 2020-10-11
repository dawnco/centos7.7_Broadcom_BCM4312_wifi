
## centos7.8 安装 Broadcom BCM4312 芯片wifi驱动

# 联想G450 安装后没有wifi驱动,网上找到的方案
# 驱动和 patch文件 在 file目录

参考
https://programmer.help/blogs/install-broadcom-bcm4312-wireless-card-driver-on-centos-7.7.html
https://my.oschina.net/itblog/blog/1023484

查看网卡芯片
```
[user@host ~]$ lspci | grep Broadcom
04:00.0 Network controller: Broadcom Limited BCM4312 802.11b/g LP-PHY (rev 01)
07:00.0 Ethernet controller: Broadcom Limited NetLink BCM5906M Fast Ethernet PCI Express (rev 02)
```

下载网卡芯片驱动
```
https://www.broadcom.com/site-search?filters[pages][Content_Type][type]=and&filters[pages][Content_Type][values][]=Downloads&page=1&per_page=10&q=BCM4312
```

安装必要软件
```
yum install kernel-headers kernel-devel gcc patch
```

```
cd /usr/local/src/hybrid-wl/
patch -p1 < ../wl-kmod-01_kernel_4.7_IEEE80211_BAND_to_NL80211_BAND.patch
patch -p1 < ../wl-kmod-02_kernel_4.8_add_cfg80211_scan_info_struct.patch
patch -p1 < ../wl-kmod-03_fix_kernel_warnings.patch
patch -p1 < ../wl-kmod-04_kernel_4.11_remove_last_rx_in_net_device_struct.patch
patch -p1 < ../wl-kmod-05_kernel_4.12_add_cfg80211_roam_info_struct.patch
```

```
cd /usr/local/src/hybrid-wl/
sed -i 's/ >= KERNEL_VERSION(3, 11, 0)/ >= KERNEL_VERSION(3, 10, 0)/' src/wl/sys/wl_cfg80211_hybrid.c
sed -i 's/ >= KERNEL_VERSION(3, 15, 0)/ >= KERNEL_VERSION(3, 10, 0)/' src/wl/sys/wl_cfg80211_hybrid.c
sed -i 's/ < KERNEL_VERSION(3, 18, 0)/ < KERNEL_VERSION(3, 9, 0)/' src/wl/sys/wl_cfg80211_hybrid.c
sed -i 's/ >= KERNEL_VERSION(4, 0, 0)/ >= KERNEL_VERSION(3, 10, 0)/' src/wl/sys/wl_cfg80211_hybrid.c
sed -i 's/ < KERNEL_VERSION(4,2,0)/ < KERNEL_VERSION(3, 9, 0)/' src/wl/sys/wl_cfg80211_hybrid.c
sed -i 's/ >= KERNEL_VERSION(4, 7, 0)/ >= KERNEL_VERSION(3, 10, 0)/' src/wl/sys/wl_cfg80211_hybrid.c
sed -i 's/ >= KERNEL_VERSION(4, 8, 0)/ >= KERNEL_VERSION(3, 10, 0)/' src/wl/sys/wl_cfg80211_hybrid.c
sed -i 's/ >= KERNEL_VERSION(4, 11, 0)/ >= KERNEL_VERSION(3, 10, 0)/' src/wl/sys/wl_cfg80211_hybrid.c
sed -i 's/ < KERNEL_VERSION(4, 12, 0)/ < KERNEL_VERSION(3, 9, 0)/' src/wl/sys/wl_cfg80211_hybrid.c
sed -i 's/ >= KERNEL_VERSION(4, 12, 0)/ >= KERNEL_VERSION(3, 10, 0)/' src/wl/sys/wl_cfg80211_hybrid.c
sed -i 's/ <= KERNEL_VERSION(4, 10, 0)/ < KERNEL_VERSION(3, 9, 0)/' src/wl/sys/wl_linux.c

```

```
modprobe -r bcm43xx
modprobe -r b43
modprobe -r b43legacy
modprobe -r ssb
modprobe -r bcma
modprobe -r brcmsmac
modprobe -r ndiswrapper
```

```
cp -vi /usr/local/src/hybrid-wl/wl.ko /lib/modules/`uname -r`/extra/
```



```
vim /etc/modprobe.d/blacklist.conf
```
输入以下内容

```
blacklist bcm43xx
blacklist b43
blacklist b43legacy
blacklist bcma
blacklist brcmsmac
blacklist ssb
blacklist ndiswrapper
```


```
vim /etc/sysconfig/modules/kmod-wl.modules
```
输入以下内容
```
#!/bin/bash

for M in lib80211 cfg80211 wl; do
    modprobe $M &>/dev/null
done
```



