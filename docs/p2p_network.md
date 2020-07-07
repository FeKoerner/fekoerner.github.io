# P2P Network

!!! summary
    This article explains how to build a P2P network.

Give your clients the following ip addresses:

* Client 1: IP `10.0.0.1`
* Client 2: IP `10.0.0.2`
* Subnet Mask: 255.255.255.252 :arrow_right: Only to clients possible 
    * [IP Calculator](http://jodies.de/ipcalc?host=10.1.6.0&mask1=30&mask2=)

!!! warning
    code needs to be verified

## /etc/network/interfaces
```
auto enp2s0
iface enp2s0 inet static
        address 10.0.0.1
        netmask 30
```

!!! tip
    Change Jumbo Frame size for better performance. Not adviced if files of a small are used.
