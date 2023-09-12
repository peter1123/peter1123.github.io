为了定制openwrt的firmware，可以从官网提供的链接完成
https://firmware-selector.openwrt.org/?version=22.03.5&target=ramips%2Fmt7621&id=xiaomi_mi-router-ac2100

Installed Packages
···
base-files busybox ca-bundle dnsmasq-full dropbear firewall4 fstools kmod-gpio-button-hotplug kmod-leds-gpio kmod-mt7603 kmod-mt7615-firmware kmod-mt7615e kmod-nft-offload libc libgcc libustream-wolfssl logd mtd netifd nftables odhcp6c odhcpd-ipv6only opkg ppp ppp-mod-pppoe procd procd-seccomp procd-ujail uboot-envtools uci uclient-fetch urandom-seed urngd wpad-wolfssl	
···

Script to run on first boot (uci-defaults)
···
# Beware! This script will be in /rom/etc/uci-defaults/ as part of the image.
# Uncomment lines to apply:
#
wlan_name="OpenWrt"
wlan_password="12345678"
#
root_password="12345678"
lan_ip_address="192.168.0.1"

pppoe_username="1234567890"
pppoe_password="123456"

# log potential errors
exec >/tmp/setup.log 2>&1

if [ -n "$root_password" ]; then
  (echo "$root_password"; sleep 1; echo "$root_password") | passwd > /dev/null
fi

# Configure LAN
# More options: https://openwrt.org/docs/guide-user/base-system/basic-networking
if [ -n "$lan_ip_address" ]; then
  uci set network.lan.ipaddr="$lan_ip_address"
  uci commit network
fi

# Configure WLAN
# More options: https://openwrt.org/docs/guide-user/network/wifi/basic#wi-fi_interfaces
if [ -n "$wlan_name" -a -n "$wlan_password" -a ${#wlan_password} -ge 8 ]; then
  uci set wireless.@wifi-device[0].disabled='0'
  uci set wireless.@wifi-iface[0].encryption='psk2'
  uci set wireless.@wifi-iface[0].ssid="$wlan_name"
  uci set wireless.@wifi-iface[0].key="$wlan_password"
  uci commit wireless
fi

# Configure PPPoE
# More options: https://openwrt.org/docs/guide-user/network/wan/wan_interface_protocols#protocol_pppoe_ppp_over_ethernet
if [ -n "$pppoe_username" -a "$pppoe_password" ]; then
  uci set network.wan.proto=pppoe
  uci set network.wan.username="$pppoe_username"
  uci set network.wan.password="$pppoe_password"
  uci commit network
fi

echo "All done!"

···
