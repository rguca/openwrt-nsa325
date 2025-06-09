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
