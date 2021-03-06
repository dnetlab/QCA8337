# QCA8337
QCA8337 VLAN and Switch

## Webcgi源码树
	--files
		--002-ct_ntwk.rule (多子网防火墙相关脚本，由net-wall初始化)
		--004-ct_ipsec.rule (ipsec相关防火墙脚本，由net-wall初始化)
		--005-ct_fw.rule (traffic rule相关实现部分，包含多子网部分，由net-wall初始化)
		--cgi_reload.sh (部分cgi初始化入口，包含switch，network，firewall)
		--ct_default.sh (系统一些默认值，由/etc/init.d/boot中调用)
		--ct_ipsec.sh (IPSec配置与防火墙实现部分)
		--ct_ntwk.sh (子网三层接口与子网DHCP服务器初始化部分)
		--ct_wan_event.sh (WAN口获取到IP之后调用)
	--src
		--firewall.c
			1.block services设置部分，使用/www/cgi-bin/firewall.sh restart初始化（DNI系统自带）
			2.traffic rule设置部分，使用/usr/sbin/cgi_reload.sh firewall_config做初始化
		--ipsec.c
			1.IPSec增，删，改设置部分，直接使用/etc/init.d/ipsec restart初始化
		--vlan.c
			1.vlan与pvid设置部分，使用/usr/sbin/cgi_reload.sh switch_config初始化
		--network.c (多子网CGI部分，以及多子网DHCP保留地址分配)
			1.LAN1配置部分，使用/etc/init.d/net-lan restart初始化
			2.其他子网，使用/usr/sbin/cgi_reload.sh network_config初始化
	--Makefile
	
## DNI系统核心初始化部分流程
	VLAN，逻辑接口，LAN与WAN协议
	
### 主要关注
		/etc/rc.d/S20opmode自启动脚本
	相关脚本，DNI封装的一些操作
		/lib/cfgmgr/cfgmgr.sh
		/lib/cfgmgr/enet.sh
		/lib/cfgmgr/opmode.sh
		/etc/init.d/net-lan
		/etc/init.d/net-wan
		
### 主要流程:
		1.S20opmode -> start-> start_stage0 -> op_create_brs_and_vifs(/lib/cfgmgr/opmode.sh)
											-> /etc/init.d/net-lan normal mode (子网IP，MAC，DHCPD，DNS等配置)
											-> /etc/init.d/net-wan normal mode (网络配置DHCP,PPPoE,Static等)
	
		/lib/cfgmgr/opmode.sh : op_create_brs_and_vifs -> normal_create_brs_and_vifs -> normal_multi_lan_vifs(创建多子网三层接口)
																					 -> normal_multi_wan_vifs(创建多WAN口三层接口)
																					 -> sw_configvlan "normal" (/lib/cfgmgr/enet.sh)
		/lib/cfgmgr/enet.sh : sw_configvlan_normal -> sw_config_ct_vlan (写VLAN配置config转uci)
												   -> sw_config_ct_port (写PVID配置config转uci)
												   -> sw_config_ct_port_fix (PVID修正,使用ssdk命令设置)

## Reference

* https://blog.csdn.net/dreamflyliwei9/article/details/46352807
