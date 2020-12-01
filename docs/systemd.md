# Systemd

!!! Summary
    This article explains the use of systemd.

## systemd-timesyncd
Systemd jobs can be triggert by time. To ensure that the system uses the correct time the service `systemd-timesyncd` can be used.
Configuration is done in the file   
`/etc/systemd/timesyncd.conf`

```
[Time]
NTP=ptbtime1.ptb.de ptbtime2.ptb.de ptbtime3.ptb
FallbackNTP=0.pool.ntp.org 1.pool.ntp.org 0.de.pool.ntp.org
```

In this example for the primary NTP-Server the NTP-Server [Physikalisch-Technischen Bundesanstallt](https://www.ptb.de/cms/ptb/fachabteilungen/abtq/gruppe-q4/ref-q42/zeitsynchronisation-von-rechnern-mit-hilfe-des-network-time-protocol-ntp.html) is used and as fallback the [Poolserver of the NTP pool project](https://www.ntppool.org/en/).

After a reboot of the service `systemd-timesyncd.service` the status can be shown.

```
user@system # timedatectl timesync-status
       Server: 192.53.103.108 (ptbtime1.ptb.de)
Poll interval: 1min 4s (min: 32s; max 34min 8s)
         Leap: normal
      Version: 4
      Stratum: 1
    Reference: PTB
    Precision: 1us (-20)
Root distance: 167us (max: 5s)
       Offset: +2.969ms
        Delay: 10.206ms
       Jitter: 0
 Packet count: 1
    Frequency: +28.083ppm
```

**Troubleshooting:** run `timedatectl set-ntp true`

## Timer

The following examples schedules `monitor.service` to be run with a corresponding timer called `monitor.timer` every minute.

`/etc/systemd/system/monitor.timer`
```
[Unit]
Description=Run every minute

[Timer]
OnCalendar=*:0/1
# run every minute
Persistent=false
# Ausführung verpasster Zeiten

[Install]
WantedBy=timers.target
```

`monitor.timer` activates `monitor.service`

`/etc/systemd/system/monitor.service`
```
[Unit]
Description=Run every Minute

[Service]
Type=oneshot
ExecStart=/home/user/run.sh

[Install]
WantedBy=multi-user.target
```

With the command `systemctl list-timers` it can be checked if the timer is activated.

## Parameter

### OnCalender
Usage of parameter **OnCalender**
```
    minutely → *-*-* *:*:00
      hourly → *-*-* *:00:00
       daily → *-*-* 00:00:00
     monthly → *-*-01 00:00:00
      weekly → Mon *-*-* 00:00:00
      yearly → *-01-01 00:00:00
   quarterly → *-01,04,07,10-01 00:00:00
semiannually → *-01,07-01 00:00:00

```
Usage examples of parameter **OnCalender**
```
  Sat,Thu,Mon..Wed,Sat..Sun → Mon..Thu,Sat,Sun *-*-* 00:00:00
      Mon,Sun 12-*-* 2,1:23 → Mon,Sun 2012-*-* 01,02:23:00
                    Wed *-1 → Wed *-*-01 00:00:00
           Wed..Wed,Wed *-1 → Wed *-*-01 00:00:00
                 Wed, 17:48 → Wed *-*-* 17:48:00
Wed..Sat,Tue 12-10-15 1:2:3 → Tue..Sat 2012-10-15 01:02:03
                *-*-7 0:0:0 → *-*-07 00:00:00
                      10-15 → *-10-15 00:00:00
        monday *-12-* 17:00 → Mon *-12-* 17:00:00
  Mon,Fri *-*-3,1,2 *:30:45 → Mon,Fri *-*-01,02,03 *:30:45
       12,14,13,12:20,10,30 → *-*-* 12,13,14:10,20,30:00
            12..14:10,20,30 → *-*-* 12..14:10,20,30:00
  mon,fri *-1/2-1,3 *:30:45 → Mon,Fri *-01/2-01,03 *:30:45
             03-05 08:05:40 → *-03-05 08:05:40
                   08:05:40 → *-*-* 08:05:40
                      05:40 → *-*-* 05:40:00
     Sat,Sun 12-05 08:05:40 → Sat,Sun *-12-05 08:05:40
           Sat,Sun 08:05:40 → Sat,Sun *-*-* 08:05:40
           2003-03-05 05:40 → 2003-03-05 05:40:00
 05:40:23.4200004/3.1700005 → *-*-* 05:40:23.420000/3.170001
             2003-02..04-05 → 2003-02..04-05 00:00:00
       2003-03-05 05:40 UTC → 2003-03-05 05:40:00 UTC
                 2003-03-05 → 2003-03-05 00:00:00
                      03-05 → *-03-05 00:00:00
                     hourly → *-*-* *:00:00
                      daily → *-*-* 00:00:00
                  daily UTC → *-*-* 00:00:00 UTC
                    monthly → *-*-01 00:00:00
                     weekly → Mon *-*-* 00:00:00
    weekly Pacific/Auckland → Mon *-*-* 00:00:00 Pacific/Auckland
                     yearly → *-01-01 00:00:00
                   annually → *-01-01 00:00:00
                      *:2/3 → *-*-* *:02/3:00
```