# 1.0 Static IP Address on Qubes OS
## 1.1 Introduction
This guide is intended to set up a static IP address and a cloned MAC address on your operating system **Qubes**. Since the qube ***'sys‑net'*** utilizes the default ***'default‑dvm'***, every time sys‑net reboots it will erase all of your settings, because you are connecting to your network via ***'sys‑net'***. The goal is to block, on your modem, all new connections that could allow a threat actor to access your network and to permit only the trusted one to connect, this cannot be achieved using the default configuration provided by the qube, so to accomplish it we need to set up a static IP address and a MAC address that is not the real one, allowing us to tell our modem to trust the cloned MAC address and the static IP address without compromising privacy and security.

## 1.2 Setting UP
Locate the template used by the ***'sys‑net'*** qube. By default it runs on the ***'default‑dvm'*** template, which should be based on a fedora template.
Open a terminal in ***'dom0'*** and run the following command to launch a root shell inside the ***'default‑dvm'*** qube:
```
myqubes@dom0:~$ qvm-run -u root default-dvm xfce4-terminal
```

We need to edit the **'/rw/config/rc.local'** file, it is a script that runs automatically each time a qube boot up. In this case, we want it to execute a predefined set of commands whenever the qube starts.
```
bash-5.2# sudo nano /rw/config/rc.local
```

Inside the following script we need to change the following variables: 
- `{$SSID}`: Replace with the name of your home network (SSID).
- `{$MAC-ADDRESS}`: Write a fake MAC address. 
- `{$STATIC-PRIVATE-ADDRESS}`: Replace with your private IP address.
- `{$SUBNET}`: Replace it with your subnet typically is '/24' (equivalent to  255.255.255.0).
- `{$GATEWAY}`: Replace with the IP address of your modem.
- `{$DNS1}` & `{$DNS2}`: Write the DNS server you wan to use. If you want to use the DNS provided by your modem, just enter the modem’s IP address.

```shell
sudo touch "/etc/NetworkManager/system-connections/{$SSID}.nmconnection"
echo """
[connection]
id={$SSID}
type=wifi
interface-name=wls6

[wifi]
mode=infrastructure
ssid={$SSID}
cloned-mac-address={$MAC-ADDRESS XX:XX:XX:XX:XX:XX}

[wifi-security]
auth-alg=open
key-mgmt=wpa-psk

[ipv4]
method=manual
address={$STATIC-PRIVATE-ADDRESS}/{$SUBNET}
gateway={$GATEWAY}
dns={$DNS1};{$DNS2};

[ipv6]
addr-gen-mode=stable-privacy
method=auto
""" | sudo tee -a "/etc/NetworkManager/system-connections/{$SSID}.nmconnection"

sudo chown root:root "/etc/NetworkManager/system-connections/{$SSID}.nmconnection"
sudo chmod 600 "/etc/NetworkManager/system-connections/{$SSID}.nmconnection"
sudo nmcli connection reload
sudo nmcli connection up {$SSID}
```

<details>
<summary>Here is a complete example of the script:</summary>

```shell
sudo touch "/etc/NetworkManager/system-connections/HomeNetwork.nmconnection"
echo """
[connection]
id=HomeNetwork
type=wifi
interface-name=wls6

[wifi]
mode=infrastructure
ssid=HomeNetwork
cloned-mac-address=c6:0a:40:06:a2:17

[wifi-security]
auth-alg=open
key-mgmt=wpa-psk

[ipv4]
method=manual
address=10.10.27.50/24
gateway=10.10.27.1
dns=10.10.27.1;10.10.27.1;

[ipv6]
addr-gen-mode=stable-privacy
method=auto
""" | sudo tee -a "/etc/NetworkManager/system-connections/HomeNetwork.nmconnection"

sudo chown root:root "/etc/NetworkManager/system-connections/HomeNetwork.nmconnection"
sudo chmod 600 "/etc/NetworkManager/system-connections/HomeNetwork.nmconnection"
sudo nmcli connection reload
```

</details>

Now shut down the ***'default‑dvm'*** template and restart the ***'sys‑net'*** qube. After the reboot, when it connects to your home network it will automatically use the static IP address you configured. 

## 1.3 Check out

To verify that the static IP address you configured is being used, open a terminal in ***'sys‑net'*** as the root user from a ***'dom0'*** terminal:
```
myqubes@dom0:~$ qvm-run -u root sys-net xfce4-terminal
```

Run `ip a` to verify that the private IP address has been assigned correctly:

```shell
bash-5.2# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens7: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    altname enp0s7
    altname enxd843aea4ff14
3: wls6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff permaddr yy:yy:yy:yy:yy:yy
    altname wlp0s6
    altname wlxdc97baf7ec07
    inet 10.10.27.50/24 brd 10.10.27.255 scope global noprefixroute wls6
       valid_lft forever preferred_lft forever
4: vif5.0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group 2 qlen 1000
    link/ether fe:ff:ff:ff:ff:ff brd ff:ff:ff:ff:ff:ff
    inet 10.138.26.139/32 scope global vif5.0
       valid_lft forever preferred_lft forever
    inet6 fe80::fcff:ffff:feff:ffff/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
```


----

