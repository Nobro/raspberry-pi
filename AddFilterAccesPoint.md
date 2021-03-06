#Create raspberry pi model B with TL-WN727N wifi usb adaptor - add filter acces point
##OS Rasbian Linux raspberrypi 3.6.11+

* * * *

###Just put the 2 tutorials in one for my wifi adaptor and a model to add more domains to the DNS ban list
###Links:

[http://learn.adafruit.com/setting-up-a-raspberry-pi-as-a-wifi-access-point](http://learn.adafruit.com/setting-up-a-raspberry-pi-as-a-wifi-access-point)

[http://learn.adafruit.com/raspberry-pi-as-an-ad-blocking-access-point](http://learn.adafruit.com/raspberry-pi-as-an-ad-blocking-access-point)

[http://ubuntuforums.org/archive/index.php/t-1666714.html](http://ubuntuforums.org/archive/index.php/t-1666714.html)

* * * *

sudo apt-get update

sudo apt-get upgrade

sudo apt-get install -y hostapd dnsmasq dnsutils

sudo nano /etc/hostapd/hostapd.conf

~~~

interface=wlan0

driver=nl80211

ssid=SSID_Name

hw_mode=g

channel=6

macaddr_acl=0

auth_algs=1

ignore_broadcast_ssid=0

wpa=2

wpa_passphrase=Your_Password

wpa_key_mgmt=WPA-PSK

wpa_pairwise=TKIP

rsn_pairwise=CCMP

~~~

sudo nano /etc/default/hostapd

~~~

DAEMON_CONF="/etc/hostapd/hostapd.conf"

~~~

sudo nano /etc/sysctl.conf

~~~

net.ipv4.ip_forward=1

~~~

sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT

sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"

sudo service hostapd start

sudo update-rc.d hostapd enable

sudo nano /etc/dnsmasq.d/dnsmasq.custom.conf

~~~

interface=wlan0

dhcp-range=wlan0,192.168.42.10,192.168.42.50,2h

# Gateway

dhcp-option=3,192.168.42.1

# DNS

dhcp-option=6,192.168.42.1

dhcp-authoritative

~~~

sudo nano /etc/resolv.conf

~~~

nameserver 192.168.42.49

nameserver 8.8.8.8

nameserver 8.8.4.4

~~~

sudo nano /usr/local/bin/dnsmasq_ad_list.sh

~~~

#!/bin/bash
 
ad_list_url="http://pgl.yoyo.org/adservers/serverlist.php?hostformat=dnsmasq&showintro=0&mimetype=plaintext"
pixelserv_ip="192.168.42.49"
ad_file="/etc/dnsmasq.d/dnsmasq.adlist.conf"
temp_ad_file="/etc/dnsmasq.d/dnsmasq.adlist.conf.tmp"
 
curl $ad_list_url | sed "s/127\.0\.0\.1/$pixelserv_ip/" > $temp_ad_file
 
if [ -f "$temp_ad_file" ]
then
        #sed -i -e '/www\.favoritesite\.com/d' $temp_ad_file #here you take out domains from the block list
        #sed -i -e '$a\address=/www\.badsites\.com/' $temp_ad_file #here you add more domains to the block list
        mv $temp_ad_file $ad_file
else
        echo "Error building the ad list, please try again."
        exit
fi
 
~~~

service dnsmasq restart

sudo chmod +x /usr/local/bin/dnsmasq_ad_list.sh

sudo crontab -e

~~~

@weekly /usr/local/bin/dnsmasq_ad_list.sh

~~~

sudo curl -o /usr/local/bin/pixelserv http://proxytunnel.sourceforge.net/files/pixelserv.pl.txt

sudo chmod 755 /usr/local/bin/pixelserv

sudo nano /usr/local/bin/pixelserv

~~~

$sock = new IO::Socket::INET ( LocalHost => '192.168.42.49',

~~~

sudo nano /etc/network/interfaces

~~~

auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
auto wlan0
iface wlan0 inet static
        address 192.168.42.1
        netmask 255.255.255.0
        network 192.168.42.0
        broadcast 192.168.42.255
        post-up ip addr add dev wlan0 192.168.42.49/24
        pre-down ip addr del dev wlan0 192.168.42.49/24

up iptables-restore < /etc/iptables.ipv4.nat

~~~

sudo nano /etc/init.d/pixelserv

~~~

### BEGIN INIT INFO
# Provides: pixelserv
# Required-Start: $network
# Required-Stop: $network
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: pixelserv server for ad blocking
# Description: Server for serving 1x1 pixels
### END INIT INFO
 
case "$1" in
   start)
     echo "pixelserv: starting"
     /usr/local/bin/pixelserv &
     ;;
   stop)
     echo "pixelserv: stopping"
     killall pixelserv
     ;;
   *)
     echo "Usage: service $0 {start|stop}"
     exit 1
     ;;
esac
 
exit 0

~~~

sudo chmod 744 /etc/init.d/pixelserv

sudo update-rc.d pixelserv defaults

sudo reboot
