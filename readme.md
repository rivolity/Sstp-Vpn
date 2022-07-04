
## # How to configure a SSTP VPN in Debian/Ubuntu
Connect to VPN using SSTP

## Installation of the sstp-client

Add the following PPA (Personal Package Archive) for Debian/Ubuntu (install the add-apt-repository application if you don't have it with `sudo apt-get install software-properties-common`)

```
sudo add-apt-repository ppa:eivnaes/network-manager-sstp
```

and install the SSTP client

```
sudo apt-get install sstp-client
```

OPTIONAL: if your use the Network Manager utility you can also install the plugin for SSTP VPN as

```
sudo apt-get install network-manager-sstp
```
## Configuration of connection Profile
You can use the CLI to do what nm-applet does behind the backend:
### Create a nmconnection file
Create a nm connection file in `/etc/NetworkManager/system-connections/NAME_OF_CONNECTION.nmconnection`

in our example, I have named my profile as below.
```
/etc/NetworkManager/system-connections/VPNCLIENT.nmconnection
```
### Configuration of Profile
the content of your profile should look like that.
you need to change the **user**, the **gateway** and the **password**.
```
[connection]
id=VPNCLIENT
uuid=2815492f-7e56-435e-b2e9-246bd7cdc664
type=vpn
autoconnect=false
permissions=
timestamp=1652878614

[vpn]
gateway=Gate-Way-Or-IP(exemple > con.entreprise.com or IP)
ignore-cert-warn=yes
password-flags=0
refuse-chap=yes
refuse-eap=yes
refuse-pap=yes
user=YOUR-USER-NAME
service-type=org.freedesktop.NetworkManager.sstp
connection-type=password

[vpn-secrets]
password=YOUR-PASSWORD

[ipv4]
dns-search=
method=auto

[ipv6]
addr-gen-mode=stable-privacy
dns-search=
method=auto

[proxy]
```
## Configuration of NetworkManager
### Create a Conf file
This configuration file is meant to manage your **network interface**.
Create the conf file as below
```/etc/NetworkManager/conf.d/10-globally-managed-devices.conf```  with the following values

```
[keyfile]
unmanaged-devices=none
```
In that way the NetworkManager will manage your interface.
### Configurate the NetworkManager in Netplan
Adding the NetworkManager to Netplan.

In `/etc/netplan/01-netcfg.yaml`, or `/etc/netplan/50-cloud-init.yaml`, or (in my case) `/etc/netplan/00-installer-config.yaml` (it may be a different name but it should be the only file located in that directory), add renderer: NetworkManager after network:

In my example, I've added the **renderer** at the end of the configuration, in network section.
```
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3: #you may have a different name here
      dhcp4: true
  version: 2
  renderer: NetworkManager
```
## Restart Services

#### Apply the new netplan yaml configuration.

    sudo netplan apply

#### Restart network manager

    sudo service NetworkManager restart

## Load VPN connection
#### List connection

    nmcli con
  
  The result of this command should be similar to this:
  
  ```
 user@user:~$ sudo nmcli con
NAME            UUID                                  TYPE      DEVICE 
netplan-enp0s3  1eef7e45-3b9d-3043-bee3-fc5925c90273  ethernet  enp0s3
VPNCLIENT       2815492f-7e56-435e-b2e9-246bd7cdc664  vpn       --     
 ```
  
#### Load VPN connection profile

    sudo nmcli con up id VPNCLIENT
  The result of this command should look like that:
  
  ```
user@user:~$ sudo nmcli con up id VPN
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/2)
 ```

By typing `ifconfig`, you should see a new Network Interface `PPP0` at the end of the console output.
```
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::a00:27ff:fef0:a13c  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:f0:a1:3c  txqueuelen 1000  (Ethernet)
        RX packets 13443  bytes 1447523 (1.4 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 9217  bytes 1982243 (1.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 20100  bytes 2605349 (2.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 20100  bytes 2605349 (2.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ppp0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.10.10.10  netmask 255.255.255.255  destination 10.10.10.18
        ppp  txqueuelen 3  (Point-to-Point Protocol)
        RX packets 11  bytes 408 (408.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12  bytes 172 (172.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

#### Unload the VPN connection profile

    sudo nmcli con down id VPNCLIENT
the result of this command

    user@user:~$ sudo nmcli con down id VPNCLIENT
    Connection 'VPNCLIENT' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/2)
