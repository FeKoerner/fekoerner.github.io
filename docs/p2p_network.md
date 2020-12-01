# P2P Network

!!! summary
    This article explains how to build a P2P network.

Give your clients the following ip addresses:

* Client 1: IP `10.0.0.1`
* Client 2: IP `10.0.0.2`
* Subnet Mask: 255.255.255.252 :arrow_right: Only two clients possible 
    * [IP Calculator](http://jodies.de/ipcalc?host=10.1.6.0&mask1=30&mask2=)

## /etc/network/interfaces
```
auto <interface>
iface <interface> inet static
        address 10.0.0.1
        netmask 30
        mtu 9000
```

!!! tip
    Change Jumbo Frame size for better performance. Not adviced if files of a small size are used.
