#Raspbian static network config for connecting to a hidden wireless AP

sudo vim /etc/network/interfaces

~~~

auto lo
iface lo inet loopback

iface eth0 inet dhcp

auto wlan0
allow-hotplug wlan0
   iface wlan0 inet static
        wpa-scan-ssid 1
        wpa-ap-scan 1
        wpa-key-mgmt WPA-PSK
        wpa-proto RSN WPA
        wpa-pairwise CCMP TKIP
        wpa-group CCMP TKIP
        wpa-ssid "SSID"
        wpa-psk "Password"
        address 192.168.1.6
        netmask 255.255.255.0
        gateway 192.168.1.1

~~~

//test signal power with sudo apt-get install wavemon

sudo wavemon
