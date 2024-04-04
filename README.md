# Immortalwrt
Setup a new router with Immortalwrt with 5g modem

This guide is applicable only for freshly building a Immortalwrt install for the first time ;  If you are already on Immortalwrt, then you need to skip to last step , where 
upgrade new stable version from previous stable version


step 1 :Install the latest stable realease from with out and rc or release candidates for thev2  architecture

https://downloads.immortalwrt.org/releases/   

for Wrt1900acs v2 version, it will be under   mvebu/cortexa9/

Everytime we flash full image, Always flash it first with the factory image of Linksys wrt1900acs

https://downloads.linksys.com/support/assets/firmware/FW_WRT1900ACSV2_2.0.3.201002_prod.img 

AFter flashed with Linksys OEM firmware , then again flash the Immortalwrt now to have clean install of immortalwrt 
use default password "admin" .

Then connectivity --> manual update --> upload the full Image of immortalwart    "https://downloads.immortalwrt.org/releases/23.05.2/targets/mvebu/cortexa9/immortalwrt-23.05.2-mvebu-cortexa9-linksys_wrt1900acs-squashfs-factory.img"

install one with squashfs - as it includes recovery partiion.

after its flashed . install the packages only from the routers/system/software section - as it installs cleanly.

If you are in r00ter or other firmwares with ssh access , the follow this steps to go back to oem firmware/stock firmware first.

WRT1900ACS V2:
--------------

cd /tmp && opkg update && opkg install wget && wget https://downloads.linksys.com/downloads/firmware/FW_WRT1900ACSV2_2.0.2.188405_prod.img
Finally, run this command to force it to upgrade

For V2:
----------

cd /tmp && sysupgrade -F -n -v FW_WRT1900ACSV2_2.0.2.188405_prod.img
Obviously just pay attention to whether it's V1 or V2 and it should work very smoothly after flashing the final command. Hope it works for you



Required Packages:
---------------------

1. install modemmanager - its connecting natively and its very stable than the r00ter
      optional : if you don't want modemmanager , then you can install quectl-cm - which is connection manager for quectel modems
2.Install modemband  -- to select 5g/4g bands and band locking
3.install sms-tool  - to send AT commands
4.isntall 3ginfo   - to display 5g/4g connection information and signal quality etc
4.install mbim-cli
5.install qmi-tools /qmi-cli /uqmi-cli
6.Install UPX package to compress the sing-box binary
6.install sing-box
7.Install kmod-tun
8.install openvpn-mbedtls
9.install kmod-wireguard


Configure Modemmananger:
--------------------------

Configure modemmanger for 5g connectivity 
  1. Follow this step     "https://radenku.com/cara-setting-modemmanager-di-openwrt/"

       steps:
     1. What about ModemManager?
ModemManager is an open source Linux program that functions to connect modems, 2G/3G/4G & 5G mobile devices, USB dongles, etc. ModemManager can control modems with various modem protocols such as QMI/RMNET, MBIM, AT Command, etc.

In OpenWRT itself there are other tools for connecting to modems such as PPP protocol, QMI Cellular, MBIM Cellular, 3G/GPRS/EV-DO, NCM etc.

It's just that each protocol is only for a specific modem, when you have 2 modems with different protocols, then the way to connect is according to each protocol.

openwrt-mobile-broadband
In contrast to Modem Manager which can handle all PPP, QMI, MBIM and NCM connections in one application. In Openwrt ModemManager has been merged into the official repository since firmware 19.07.

Even though the modem manager already exists in OpenWRT 19.07, the repository doesn't yet have an OpenWRT LuCI ModemManager, so you have to install it manually from the repository snapshot. To set up ModemManager Openwrt, you can follow the tutorial below.

2. Modem Support ModemManager
ModemManager supports several modem protocols such as QMI, MBIM, PPP etc. From 2G, 3G, 4G modems, there are even 5G modems that are supported by ModemManager.

Here are some modems that I have tried to connect using ModemManager

HP Lt4220 (Telit Ln940) Mode QMI & MBIM
Dell Dw5821e (Telit Ln960) QMI & MBIM mode
Sierra Em7430 (Dell Dw5816e) QMI & MBIM mode
Sierra Em7455 (Dell Dw5811e) QMI & MBIM Mode
Quectel Ep06-e & Em06-e QMI & MBIM Modes
Bolt Bl100 Mode QMI
Fibocom L850-GL Mode MBIM
Huawei Me909s-821 Mode NCM
For this tutorial, we only use a few basic mmcli commands. All mmcli commands can be seen below on the modemmanager manual page at freedesktop.org .

3. Install ModemManager Openwrt
Setting-ModemManager-di-Openwrt-2022
To install OpenWRT ModemManager, you can install it directly using OPKG like installing other OpenWRT packages, you can use the terminal or LuCI , for the tutorial here I use the terminal.

First of all, make sure the router is connected to the internet first. Enter the openwrt terminal .

Then update the repository.

opkg update
Then install the required driver package

opkg install kmod-mii kmod-usb-net kmod-usb-wdm kmod-usb-net-qmi-wwan uqmi luci-proto-qmi \
kmod-usb-net-cdc-ether kmod-usb-serial-option kmod-usb-serial kmod-usb-serial-wwan qmi-utils \
kmod-usb-serial-qualcomm kmod-usb-acm kmod-usb-net-cdc-ncm kmod-usb-net-cdc-mbim umbim \
modem manager luci-proto-modem-manager usbutils
Also read: How to Install the Openwrt Package

install-modemmanager-openwrt
If all packages have been installed, reboot the router first.

After rebooting, just plug the modem into the router's USB port. We first check that the modem has been detected & all the drivers have been installed correctly.

We can check using usbutils with the following command.

lsusb && lsusb -t
In this post I will try to demonstrate using the Dell Dw5821e MBIM modem.

root@OpenWrt:~# lsusb && lsusb -t
Bus 002 Device 001: ID 1d6b:0001 Linux 5.4.154 ohci_hcd Generic Platform OHCI controller
Bus 004 Device 002: ID 0bda:8153 Realtek USB 10/100/1000 LAN
Bus 004 Device 001: ID 1d6b:0003 Linux 5.4.154 xhci-hcd xHCI Host Controller
Bus 001 Device 002: ID 413c:81d7 Dell Inc. DW5821e Snapdragon X20 LTE
Bus 001 Device 001: ID 1d6b:0002 Linux 5.4.154 ehci_hcd EHCI Host Controller
Bus 003 Device 001: ID 1d6b:0002 Linux 5.4.154 xhci-hcd xHCI Host Controller
/:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 5000M
    |__ Port 1: Dev 2, If 0, Class=, Driver=r8152, 5000M
/:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 480M
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ohci-platform/1p, 12M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-platform/1p, 480M
    |__ Port 1: Dev 2, If 0, Class=, Driver=cdc_mbim, 480M
    |__ Port 1: Dev 2, If 1, Class=, Driver=cdc_mbim, 480M
    |__ Port 1: Dev 2, If 2, Class=, Driver=option, 480M
    |__ Port 1: Dev 2, If 3, Class=, Driver=option, 480M
    |__ Port 1: Dev 2, If 4, Class=, Driver=option, 480M
    |__ Port 1: Dev 2, If 5, Class=, Driver=option, 480M
    |__ Port 1: Dev 2, If 6, Class=, Driver=, 480M
The Dell Dw5821e modem has been detected with vid:pid 413c:81d7 and the driver has been installed, namely cdc_mbim & serial option .

This may be different from other modems such as the Sierra EM7430 , Quectel EP06 E , Fibocom L850 GL or other modems. The drivers also vary depending on the modem, such as using cdc_mbim or qmi_wwan and the serial can be optional , ACM or qcserial

4. Cara Setting ModemManager Openwrt
Once the modem is confirmed to be detected in OpenWRT, then we check it in the ModemManager command using the commandmmcli.

To check the modem connected to ModemManager use the commandmmcli -L
mmcli -L
An example of a response like this, I connect 2 Dell Dw5821e and Telit Ln940 modems and two modems will be detected.

root@OpenWrt:~# mmcli -L
    /org/freedesktop/ModemManager1/Modem/ 0 [ Dell Inc. ] DW5821e Snapdragon X20 LTE 
    /org/freedesktop/ModemManager1/Modem/ 1 [ Telit ] Telit LN940 Mobile Broadband 
Then ModemManager detects the modem with

no. 0 for Dell Dw5821e
No. 1 for Telit Ln940
To find out detailed info on modem number 0 (Dell Dw5821e) use the following command

mmcli -m 0
And for detailed information on the Telit ln940 then

mmcli -m 1
Following is the detailed modem info

root@OpenWrt:~# mmcli -m 0
  -----------------------------------
  General   |                     path: /org/freedesktop/ModemManager1/Modem/ 0
           |               device id: 7ab53b615ec81ece36b7a9665316dfxyz
  -----------------------------------
  Hardware |            manufacturer: Dell Inc.
           |                   model: DW5821e Snapdragon X20 LTE
           |       firmware revision: T77W968.F1.0.0.5.2.GC.013
           |                          035
           |          carrier config: GCF
           | carrier config revision: 08E0000D
           |            h/w revision: DW5821e Snapdragon X20 LTE
           |               supported: gsm-umts, lte
           |                 current: gsm-umts, lte
           |            equipment id: xyz
  -----------------------------------
  System   |                  device: /sys/devices/platform/ff5c0000.usb/usb1/1-1
           |                 drivers: cdc_mbim, option1
           |                  plugin: dell
           |            primary port: cdc-wdm0
           |                   ports: cdc-wdm0 (mbim), ttyUSB0 (at), ttyUSB1 (at), ttyUSB2 (gps),
           |                          ttyUSB3 (qcdm), wwan0 (net)
  -----------------------------------
  Status   |          unlock retries: sim-pin2 (3)
           |                   state: disabled
           |             power state: on
           |          signal quality: 0% (cached)
  -----------------------------------
  Modes    |               supported: allowed: 3g; preferred: none
           |                          allowed: 4g; preferred: none
           |                          allowed: 3g, 4g; preferred: 4g
           |                          allowed: 3g, 4g; preferred: 3g
           |                 current: allowed: 3g, 4g; preferred: 4g
  -----------------------------------
  Bands    |               supported: utran-1, utran-4, utran-6, utran-5, utran-8, utran-9,
           |                          utran-2, eutran-1, eutran-2, eutran-3, eutran-4, eutran-5, eutran-7,
           |                          eutran-8, eutran-12, eutran-13, eutran-14, eutran-17, eutran-18,
           |                          eutran-19, eutran-20, eutran-25, eutran-26, eutran-28, eutran-29,
           |                          eutran-30, eutran-32, eutran-38, eutran-39, eutran-40, eutran-41,
           |                          eutran-42, eutran-43, eutran-46, eutran-48, eutran-66, utran-19
           |                 current: eutran-1, eutran-3, eutran-8
  -----------------------------------
  IP       |               supported: ipv4, ipv6, ipv4v6
  -----------------------------------
  3GPP     |                    imei: xyz
           |           enabled locks: fixed-dialing
  -----------------------------------
  3GPP EPS |    ue mode of operation: csps-2
  -----------------------------------
  SIM       |         primary sim path: /org/freedesktop/ModemManager1/SIM/ 0
The modem has been read, but the state is still disabled. We continue setting up the OpenWRT ModemManager interface.

Masuk ke LuCI, menu Network > Interface.

Create a new interface. For example, I call the interface mm. Select the ModemManager protocol.

Cara Setting ModemManager di Openwrt 1 | radenku
Next, modem interface, select our modem, adjust the APN, here I use the default internet.

IP type select IPv4 Only because currently operators in Indonesia do not seem to have implemented IPv6. So if IPv4 & IPv6 is selected it often fails to connect.

Cara Setting ModemManager di Openwrt 2 | radenku
Then in Firewall settings select WAN.

Cara Setting ModemManager di Openwrt 3 | radenku
Lalu save & apply.

If the modem is connected, the interface will get an IP like this.

Cara Setting ModemManager di Openwrt 4 | radenku
When checked again usemmcli -m 0then the status changes to connected

root@OpenWrt:~# mmcli -m 0
  -----------------------------------
  General   |                     path: /org/freedesktop/ModemManager1/Modem/ 0
           |               device id: 7ab53b615ec81ece36b7a9665316dfxyz
  -----------------------------------
  Hardware |            manufacturer: Dell Inc.
           |                   model: DW5821e Snapdragon X20 LTE
           |       firmware revision: T77W968.F1.0.0.5.2.GC.013
           |                          035
           |          carrier config: GCF
           | carrier config revision: 08E0000D
           |            h/w revision: DW5821e Snapdragon X20 LTE
           |               supported: gsm-umts, lte
           |                 current: gsm-umts, lte
           |            equipment id: xyz
  -----------------------------------
  System   |                  device: /sys/devices/platform/ff5c0000.usb/usb1/1-1
           |                 drivers: cdc_mbim, option1
           |                  plugin: dell
           |            primary port: cdc-wdm0
           |                   ports: cdc-wdm0 (mbim), ttyUSB0 (at), ttyUSB1 (at), ttyUSB2 (gps),
           |                          ttyUSB3 (qcdm), wwan0 (net)
  -----------------------------------
  Status   |          unlock retries: sim-pin2 (3)
           |                   state: connected
           |             power state: on
           |             access tech: lte
           |          signal quality: 9% (recent)
  -----------------------------------
  Modes    |               supported: allowed: 3g; preferred: none
           |                          allowed: 4g; preferred: none
           |                          allowed: 3g, 4g; preferred: 4g
           |                          allowed: 3g, 4g; preferred: 3g
           |                 current: allowed: 3g, 4g; preferred: 4g
  -----------------------------------
  Bands    |               supported: utran-1, utran-4, utran-6, utran-5, utran-8, utran-9,
           |                          utran-2, eutran-1, eutran-2, eutran-3, eutran-4, eutran-5, eutran-7,
           |                          eutran-8, eutran-12, eutran-13, eutran-14, eutran-17, eutran-18,
           |                          eutran-19, eutran-20, eutran-25, eutran-26, eutran-28, eutran-29,
           |                          eutran-30, eutran-32, eutran-38, eutran-39, eutran-40, eutran-41,
           |                          eutran-42, eutran-43, eutran-46, eutran-48, eutran-66, utran-19
           |                 current: eutran-1, eutran-3, eutran-8
  -----------------------------------
  IP       |               supported: ipv4, ipv6, ipv4v6
  -----------------------------------
  3GPP     |                    imei: xyz
           |           enabled locks: fixed-dialing
           |             operator id: 51011
           |           operator name: XL Axiata
           |            registration: home
  -----------------------------------
  3GPP EPS |    ue mode of operation: csps-2
           |      initial bearer path: /org/freedesktop/ModemManager1/Bearer/ 8
  -----------------------------------
  SIM       |         primary sim path: /org/freedesktop/ModemManager1/SIM/ 4
  -----------------------------------
  Bearer    |                    paths: /org/freedesktop/ModemManager1/Bearer/ 9
5. Troubleshoot
Modem state disable
If the modem has been detected inmmcli, I have created an interface but still can't connect, try rebooting the router then connecting again.

Another way to connect ModemManager is using the terminal.

First, enable the modem firstmmcli -m 0 -e(assuming modem number 0)

mmcli -m 0 -e
To connect the modem use the command mmcli -m 0 --simple-connect="apn=internet"(adjust modem & apn number).

mmcli -m 0 --simple-connect="apn=internet"
Restart the mm interface if the interface still cannot get an IP.

Unknown package “luci-proto-modemmanager”
For firmware 19.07 in the repo there is no openwrt modemmanager luci yet. The solution is to install the luci-proto-modemmanager ipk file. The ipk file can be taken from the openwrt snapshot repo at https://downloads.openwrt.org/ . Just install the ipk package file.

For the base lean/lede luci-proto-modemmanager firmware you cannot use the official IPK, you can usually use the source from Immortalwrt . And in the interface menu the protocol is not ModemManager but is called Mobile Data .

Cara Setting ModemManager di Openwrt 6 | radenku
Setting interface dari Firmware Openwrt Mod
If you use the build & mod firmware that already has ModemManager installed, then we only need to edit/create a new interface.

Masuk ke Luci menu Network -> Interface.

If there is a wwan0 interface (the example below is an interface called UWAN0) it is best to delete it and just create a new interface with the ModemManager protocol. Or the interface protocol has been changed to ModemManager.

Cara Setting ModemManager di Openwrt 5 | radenku
Dell Dw5821e modem manager no devices found in system
For Dw5821e it is not read in ModemManager & the cdc_mbim driver also does not appear, then ModemManager must be upgraded. Learn more about how to solve Dell Dw5821e not detected in ModemManager .

ModemManager cannot run after installing xmm-modem
For those using xmm-modem with ModemManager, make sure to turn off the xmm-modem. Otherwise xmm-modem will take over the eth1, eth2, wwan0 & usb0 interfaces, causing the modem not to connect.

To disable it, just edit the file/etc/config/xmm-modemchange the enable option to 0

config xmm-modem
    option device '/dev/ttyACM0'
    option apn 'internet'
    option enable '0'

done


Once done, restart modemmanager interface or reboot the router.

configure Modemband:
---------------------
2. Configure modemband package. for now the modemband did not support rm510q-gl, but you can take any of rm500 series template which is /usr/share/modemband
    and create a copy with name 2c7c0800RM510Q-gl    and refresh the modemband ui, then you can choose, and it starts working.

3.Configure  sing-box :
-------------------------

Refer url:
===========
https://habr.com/ru/articles/756178/

Imprtant Note:
--------------------
If sing-box ping works through tun interface but if the dns does not work through tun interface or you see
reality verification failed . Then its the time settings in router. I had left the default Asia/Shangahi time setting
and it keeps failing the reality verification failed. But after i set the correct timezone in router, Then the tunnel worked flawlessly


In this guide we will install the package sing box on ImmortalWrt 23.05.1

Recommended router at least 128 MB RAM ( 256 preferably ) and a memory of more than 16 MB, the way to install sing-box in RAM ( is suitable for devices with a small amount of ROM < 16 Mb ) will also be described.

Sing-Box —is a free and open-source proxy platform that allows users to bypass internet censorship and access blocked websites. It is an alternative to V2ray and XRAY. It can be used with various V2Ray clients on platforms such as Windows, macOS, Linux, Android, and iOS.

In addition to supporting Shadowsocks, Trojan, Vless, and Socks protocols, it also supports new protocols such as ShadowTLSv3,Hysteria2,Tuic and NaiveProxy.

Pay attention to this point, to install Sing-box in the router, use the builds taken for the passwall.

openwrt-passwall-build

The manual will include:

Setting sing-box
For the ImmortalWrt version 23.05.1 , You must enter the following commands:

Update the list of packages:

 opkg update
Next, we install the necessary for work sing box kernel modules and compatibility package with iptables:

 opkg install kmod-inet-diag kmod-netlink-diag kmod-tun iptables-nft
We are waiting for the installation to complete, the packages occupied about 1MB of memory.

Next, go to the installation sing box

 opkg install sing-box
The package takes about 10MB, so it cannot be installed on devices with 16 MB of ROM without additional manipulations. If the package is successfully installed, we proceed to configure the connection.

Configuring sing-box for Hysteria2+Vless+gRPC
Next, go to the configuration file, by default this /etc/sing-box/config.json, but available when installed /etc/sing-box/config.json.example need to create yourself.

/etc/sing-box/config.json
Example config.json:

config.json.example

Configuration writes in log /tmp/sing-box.log warnings and errors, raises stocks proxy on port 1080, raises the tunnel tun0 using kmod-tun, parameter "auto_route": true defines shadows as the default route, replace the value with false if this is not required.

Next, check the configuration performance:

 sing-box check -c /etc/sing-box/config.json
If everything is correct, the team will not make mistakes.

Next, we check the work of proxy:

 sing-box run -c /etc/sing-box/config.json
If everything worked, we can add sing box to start the start, for this we enter the commands:

/etc/init.d/sing-box enable
/etc/init.d/sing-box start
create a start-up service for sing box.
Create a file /etc/init.d/sing-box as follows:

#!/bin/sh /etc/rc.common
#
# Copyright (C) 2022 by nekohasekai <contact-sagernet@sekai.icu>
# 
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
# 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with this program. If not, see <http://www.gnu.org/licenses/>.
#

START=99
USE_PROCD=1

#####  ONLY CHANGE THIS BLOCK  ######
PROG=/usr/bin/sing-box 
RES_DIR=/etc/sing-box/ # resource dir / working dir / the dir where you store ip/domain lists
CONF=./config.json   # where is the config file, it can be a relative path to $RES_DIR
#####  ONLY CHANGE THIS BLOCK  ######

start_service() {
  sleep 10 
  procd_open_instance
  procd_set_param command $PROG run -D $RES_DIR -c $CONF

  procd_set_param user root
  procd_set_param limits core="unlimited"
  procd_set_param limits nofile="1000000 1000000"
  procd_set_param stdout 1
  procd_set_param stderr 1
  procd_set_param respawn "${respawn_threshold:-3600}" "${respawn_timeout:-5}" "${respawn_retry:-5}"
  procd_close_instance
  iptables -I FORWARD -o tun+ -j ACCEPT
  echo "sing-box is started!"
}

stop_service() {
  service_stop $PROG
  iptables -D FORWARD -o tun+ -j ACCEPT
  echo "sing-box is stopped!"
}

reload_service() {
  stop
  sleep 5s
  echo "sing-box is restarted!"
  start
}
We make the file executable:

chmod +x /etc/init.d/sing-box
Then add to the startup:

/etc/init.d/sing-box enable
/etc/init.d/sing-box start
Now, in order to activate the kill switch function on the router, so that the exchanged traffic only passes through the Singbox, we must do the following steps:

1.Click on Network → Interfaces, then click on the button of the Add new interface... In the opened page, we choose a name for the interface (for example, tun0) In the protocol section, we select Unmanaged and finally, in the device section, we select the desired connection, which is called nekoray-tun based on our configuration. And we create the Intended interface.

2.Click on Network → Firewall, in General Settings in the section Zones click on the button of the Add Zone In the name field, choose a name for the new zone (for exmaple, Sing_box) and set the values ​​of Input, Output and Forward equal to the accept value. and activate the Masquerading option and set the value of Covered networks equal to the interface we created. And the last change that should be applied in the zones section is related to the LAN zone Click on the edit button in the LAN zone and set the value of Allow forward to destination zones equal to the zone that we created for the Sing_box.

This setting is over :)



