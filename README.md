# Uboot

```
# turn on USB power
mw.l f1010100 0020c000
```

# HDDs
```
opkg install kmod-md-linear mdadm block-mount luci-app-hd-idle luci-app-samba4
mdadm --assemble /dev/md1 /dev/sda2 --run

# if you experience periodic disk access and disk won't sleep
# this runs ext4lazyinit
mount -o init_itable=0 /dev/sdb0 /mnt/sdb0
```

Configure:
* System -> Mount Points
* Services -> HDD Idle
* Services -> Network Shares


# USB ethernet

TP-LINK TL-UE300:
`opkg install kmod-usb-net-rtl8152`

# Samba
If mount on client is not possible, check if DNS on server is set.

To change name:
* Change hostname
* Restart avahi daemon
* Clear DNS cache on router and computer

# Mosquitto
`opkg install mosquitto`

edit `/etc/mosquitto/mosquitto.conf`

```
listener 1883 0.0.0.0
allow_anonymous true
```


# Tethering
```
opkg install kmod-usb-net-rndis

# For offline install download and install kmod packages manually
# URL example for 24.14.: https://archive.openwrt.org/releases/24.10.0/targets/kirkwood/generic/kmods/6.6.73-1-f700372eebff7e01d4ce7578cb03862b/
# kmod-mii
# kmod-usb-net
# kmod-usb-net-cdc-ether
# kmod-usb-net-rndis

```

# WireGuard VPN
```
opkg update
opkg install wireguard-tools luci-proto-wireguard

VPN_IF="vpn"
VPN_PORT="51820"
VPN_ADDR="192.168.7.1/24"
VPN_ADDR6="fd00:7::1/64"
ls
umask go=
wg genkey | tee wgserver.key | wg pubkey > wgserver.pub
wg genkey | tee wgclient.key | wg pubkey > wgclient.pub
wg genpsk > wgclient.psk

VPN_KEY="$(cat wgserver.key)"
VPN_PSK="$(cat wgclient.psk)"
VPN_PUB="$(cat wgclient.pub)"
uci rename firewall.@zone[0]="lan"
uci rename firewall.@zone[1]="wan"
uci del_list firewall.lan.network="${VPN_IF}"
uci add_list firewall.lan.network="${VPN_IF}"
uci -q delete firewall.wg
uci set firewall.wg="rule"
uci set firewall.wg.name="Allow-WireGuard"
uci set firewall.wg.src="wan"
uci set firewall.wg.dest_port="${VPN_PORT}"
uci set firewall.wg.proto="udp"
uci set firewall.wg.target="ACCEPT"
uci commit firewall
service firewall restart

uci -q delete network.${VPN_IF}
uci set network.${VPN_IF}="interface"
uci set network.${VPN_IF}.proto="wireguard"
uci set network.${VPN_IF}.private_key="${VPN_KEY}"
uci set network.${VPN_IF}.listen_port="${VPN_PORT}"
uci add_list network.${VPN_IF}.addresses="${VPN_ADDR}"
uci add_list network.${VPN_IF}.addresses="${VPN_ADDR6}"
uci -q delete network.wgclient
uci set network.wgclient="wireguard_${VPN_IF}"
uci set network.wgclient.public_key="${VPN_PUB}"
uci set network.wgclient.preshared_key="${VPN_PSK}"
uci add_list network.wgclient.allowed_ips="${VPN_ADDR%.*}.2/32"
uci add_list network.wgclient.allowed_ips="${VPN_ADDR6%:*}:2/128"
uci commit network
service network restart

iptables -D PREROUTING 1 -t nat
iptables -A PREROUTING -t nat -p udp -i rmnet_data0 --dport 51820 -j DNAT --to-destination 192.168.1.4:51820
# for debugging
# iptables -A PREROUTING -t nat -p icmp -i rmnet_data0 -j DNAT --to-destination 192.168.1.4
```
Add NAT rule:
<img width="943" alt="Bildschirmfoto 2025-06-29 um 17 03 46" src="https://github.com/user-attachments/assets/649545b8-6e73-4016-a645-137b8712aa6b" />

# sensors.sh
```
opkg install i2c-tools smartmontools
```

```
echo "CPU Temperature:" $(($(/usr/sbin/i2cget -y 0x0 0x0a 0x07)))"°C"
echo "HDD Temperature:" $(smartctl -A /dev/sda | grep Temperature | awk '{ print $10 }')"°C"
echo "HDD State:" $(smartctl -x /dev/sda | grep State | awk '{ print $3 }')
echo -n "Fanspeed: "
tacho=$(($(/usr/sbin/i2cget -y 0x0 0x0a 0x08)))
if [ $tacho -gt 0 ]; then
   echo $((60000/tacho))
else
   echo 0
fi
```

# Turn on after power failure

```
# Read value 0 = stay off, 1 = turn on
i2cget -y 0x0 0x50 0x7 b

# Set value
i2cset -y 0x0 0x50 0x7 0x01 b

```
