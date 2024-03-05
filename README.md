# Uboot

```
# turn on USB power
mw.l f1010100 0020c000
```

# HDDs
```
opkg install kmod-md-linear mdadm block-mount luci-app-hd-idle
mdadm --assemble /dev/md1 /dev/sdb2 --run
```

# Tethering
```
opkg install kmod-usb-net-rndis
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
