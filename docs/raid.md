

`mdadm --assemble --scan`

`mdadm --detail --scan >> /etc/mdadm/mdadm.conf`

mount as always

Ich vermute tatsächlich das der Debian Installer da viel übernimmt
[15.6., 14:25] Nils Winnwa: Beim erstellen von deinem RAID werden Header auf die Platten geschrieben. So kann, sobald alle Platten zusammen sind, das RAID in beliebiger Reihenfolge zusammen gebaut werden. Siehst du hier:


```
root@debian ~ # mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Fri Dec 27 17:02:27 2019
        Raid Level : raid1
        Array Size : 2929608128 (2793.89 GiB 2999.92 GB)
     Used Dev Size : 2929608128 (2793.89 GiB 2999.92 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Mon Jun 15 14:24:46 2020
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : bitmap

              Name : rescue:1
              UUID : 2e88af1c:0427f298:8b393bda:ade45222
            Events : 55529

    Number   Major   Minor   RaidDevice State
       0       8        2        0      active sync   /dev/sda2
       1       8       18        1      active sync   /dev/sdb2
```

 
Persistence: Superblock is persistent, sprich alle Geräte haben Header und können für die Wiederherstellung genutzt werden
[15.6., 14:28] Nils Winnwa: Und hier siehst du wie mdadm das Array beim booten zusammen puzzelt:


```
root@debian ~ # dmesg | grep -i raid
[    0.000000] Command line: BOOT_IMAGE=/vmlinuz-4.19.0-8-amd64 root=/dev/mapper/raid-root ro nomodeset consoleblank=0
[    0.208176] Kernel command line: BOOT_IMAGE=/vmlinuz-4.19.0-8-amd64 root=/dev/mapper/raid-root ro nomodeset consoleblank=0
[    1.887307] md/raid1:md0: active with 2 out of 2 mirrors
[    1.924807] md/raid1:md1: active with 2 out of 2 mirrors
```
