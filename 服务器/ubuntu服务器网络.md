## ubuntu服务器设置静态ip

#### 1 查看路由信息 2018

```aidl
root@zzlserver:/etc/netplan# ip route
default via 192.168.173.3 dev ens33 proto dhcp src 192.168.173.89 metric 100   ------> 此行为路由信息
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.18.0.0/16 dev br-aa9d45074e8c proto kernel scope link src 172.18.0.1 linkdown 
192.168.173.0/24 dev ens33 proto kernel scope link src 192.168.173.89 metric 100 
192.168.173.3 dev ens33 proto dhcp scope link src 192.168.173.89 metric 100 
```

#### 2 查看dns服务器信息

```aidl
root@zzlserver:~# cat /etc/resolv.conf
# This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers ---------> important point
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search .
root@zzlserver:~# 
```
看到重点信息于是

```aidl
root@zzlserver:~# resolvectl status
Global
       Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
resolv.conf mode: stub

Link 2 (ens33)
    Current Scopes: DNS
         Protocols: +DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 192.168.173.3  ---------> important point
       DNS Servers: 192.168.173.3

Link 3 (br-aa9d45074e8c)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 4 (docker0)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
root@zzlserver:~# 
```
#### 3 设置静态ip地址

```aidl
root@zzlserver:~# vim /etc/netplan/00-installer-config.yaml 


# This is the network config written by 'subiquity'
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no # 禁止获取动态ip
      addresses: [192.168.173.88/24] # 这个是ip地址的访问
      gateway4: 192.168.173.3 # 
      nameservers:
        addresses: [192.168.173.3] # dns
```
设置完成后
```aidl

root@zzlserver:~# netplan apply

** (generate:2380): WARNING **: 11:56:09.256: `gateway4` has been deprecated, use default routes instead.
See the 'Default routes' section of the documentation for more details.

** (process:2378): WARNING **: 11:56:09.496: `gateway4` has been deprecated, use default routes instead.
See the 'Default routes' section of the documentation for more details.


root@zzlserver:~# systemctl restart systemd-networkd
root@zzlserver:~# 
```

ok了
### 我是用的root用户，所有没有用到sudo