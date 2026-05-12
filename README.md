# Scirpt-Mikrotik-Router-OS7-
Script to start mikrotik 
# MikroTik RB750 Homelab Setup - RouterOS v7
# Dual WAN Load Balancing + Failover + Telegram Monitoring
# Network: 192.168.0.0/23 | Gateway: 192.168.0.1 | DHCP: 192.168.0.100-192.168.1.254

/system identity set name=homelab
/user add name=hasky password="260584" group=full

/system ntp client set enabled=yes server-addresses=a.ntp.br,b.ntp.br
/ip dns set servers=9.9.9.9,1.1.1.1 allow-remote-requests=yes

/interface bridge add name=BR-LAN
/interface bridge port add bridge=BR-LAN interface=ether3
/interface bridge port add bridge=BR-LAN interface=ether4
/interface bridge port add bridge=BR-LAN interface=ether5
/ip address add address=192.168.0.1/23 interface=BR-LAN

/ip pool add name=pool-lan ranges=192.168.0.100-192.168.1.254
/ip dhcp-server network add address=192.168.0.0/23 gateway=192.168.0.1 dns-server=9.9.9.9,1.1.1.1
/ip dhcp-server add name=dhcp-lan interface=BR-LAN address-pool=pool-lan lease-time=1h disabled=no

/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
/ip firewall nat add chain=srcnat out-interface=ether2 action=masquerade

/ip firewall mangle add chain=prerouting in-interface=BR-LAN connection-state=new dst-address-type=!local nth=2,0 action=mark-routing new-routing-mark=to_ether1 passthrough=yes
/ip firewall mangle add chain=prerouting in-interface=BR-LAN connection-state=new dst-address-type=!local nth=2,1 action=mark-routing new-routing-mark=to_ether2 passthrough=yes

/ip route add dst-address=0.0.0.0/0 gateway=ether1 routing-mark=to_ether1 distance=1 check-gateway=ping
/ip route add dst-address=0.0.0.0/0 gateway=ether2 routing-mark=to_ether2 distance=1 check-gateway=ping
/ip route add dst-address=0.0.0.0/0 gateway=ether1,ether2 distance=1 check-gateway=ping

/system script add name=wan-monitor source=":global wan1Status; :global wan2Status; :if (\$wan1Status=\"\") do={ :set wan1Status true }; :if (\$wan2Status=\"\") do={ :set wan2Status true }; :local botToken \"YOUR_BOT_TOKEN\"; :local chatId \"YOUR_CHAT_ID\"; :local url \"https://api.telegram.org/bot\$botToken/sendMessage\"; :local currentWan1 [/interface get ether1 running]; :local currentWan2 [/interface get ether2 running]; :if (\$currentWan1 != \$wan1Status) do={ :set wan1Status \$currentWan1; :local s1 \"DOWN\"; :if (\$currentWan1) do={ :set s1 \"UP\" }; :local msg (\"🔌 ether1 is now \" . \$s1); :log warning \$msg; /tool fetch url=\$url http-method=post http-data=(\"chat_id=\" . \$chatId . \"&text=\" . \$msg) http-header-field=\"content-type: application/x-www-form-urlencoded\" check-cert=no keep-result=no }; :if (\$currentWan2 != \$wan2Status) do={ :set wan2Status \$currentWan2; :local s2 \"DOWN\"; :if (\$currentWan2) do={ :set s2 \"UP\" }; :local msg (\"🔌 ether2 is now \" . \$s2); :log warning \$msg; /tool fetch url=\$url http-method=post http-data=(\"chat_id=\" . \$chatId . \"&text=\" . \$msg) http-header-field=\"content-type: application/x-www-form-urlencoded\" check-cert=no keep-result=no }"
/system scheduler add name=wan-monitor-schedule interval=10s on-event=wan-monitor start-time=startup disabled=no
