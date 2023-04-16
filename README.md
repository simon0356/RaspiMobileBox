# RaspiMobileBox
Raspberry Pi Wifi/ethernet access point over usb tehering

                 +- Android -----+     +----- RPi ----------------+
                 | USB Therering |     |                          +
(Internet)---WAN-+ DHCP server   +-USB-+               / WLAN AP  +-)))  (((-+ WLAN Client (DHCP Client)
                 |   192.168.x.x |     |   Bridge DHCP server     +
                 +---------------+     |   192.168.6.254          +           +- Ethernet switch ----+
                                       |               \ Ethernet +---RJ45----+               port 1 | ---RJ45--- + Ethernet DHCP Client
                                       |                          +           |                    2 |
                                       +--------------------------+           |                    3 |
                                                                              |                    4 |
                                                                              |                    5 |
                                                                              |                    n |
                                                                              +----------------------+

## Install dependencies 

sudo apt-get update
sudo apt-get upgrade

sudo apt-get install hostapd bridge-utils dnsmasq iptables

## Configure services

nano /etc/dhcpcd.conf
```
interface br0
static ip_address=192.168.6.254/24
denyinterfaces eth0
denyinterfaces wlan0
```

nano /etc/hostapd/hostapd.conf
```
interface=wlan0
bridge=br0
#driver=nl80211
ssid=RaspberryMobileBox
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=my_wpa_passphrase_secret
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

nano /etc/default/hostapd
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
nano /etc/dnsmasq.conf
```
interface=br0
    dhcp-range=192.168.6.5,192.168.6.30,255.255.255.0,24h
```
nano /etc/sysctl.conf
```
    net.ipv4.ip_forward=1
```

nano /etc/network/interfaces
```
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source /etc/network/interfaces.d/*

auto usb0
allow-hotplug usb0
iface usb0 inet dhcp
        post-up iptables -t nat -A POSTROUTING -o usb0 -j MASQUERADE
        post-up iptables -A FORWARD -i br0 -o usb0 -m state --state RELATED,ESTABLISHED -j ACCEPT
        post-up iptables -A FORWARD -i usb0 -o br0 -j ACCEPT

auto br0
iface br0 inet manual
bridge_ports eth0 wlan0
```
Enable services : 

systemctl unmask hostapd
systemctl enable hostapd
systemctl start hostapd
systemctl enable dnsmasq


## Sources : 
https://thepi.io/how-to-use-your-raspberry-pi-as-a-wireless-access-point/
https://insberr.github.io/pi-internet-bridge/
