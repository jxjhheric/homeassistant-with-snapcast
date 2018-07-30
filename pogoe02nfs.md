NFS服务端设置:
更改/etc/exports
```
/srv/nfs4/pogo                 192.168.1.0/24(rw,sync,no_root_squash,no_subtree_check)
```
exportfs -av
-------------------------------------------------------------------------------
更改/etc/fstab
```
/dev/root       /               nfs    noatime,errors=remount-ro 0 1
```
------------------------------------------------------------------------------
更改/etc/network/interfaces
```
#auto lo eth0   取消网络auto-configuration
```
------------------------------------------------------------------------------
uboot 设置：
```
fw_setenv net_dhcp_c '1'
fw_setenv net_dhcp_s '0'
fw_setenv net_c_ipaddr ''
fw_setenv net_c_netmask ''
fw_setenv net_c_gatewayip ''
fw_setenv net_nfs_server '192.168.1.204'
fw_setenv net_nfs_path   '/srv/nfs4/pogo'
fw_setenv net_nfs_dtb    'boot/dts/kirkwood-pogo_e02.dtb'
fw_setenv net_nfs_initrd 'boot/uInitrd'
fw_setenv net_nfs_kernel 'boot/uImage'

fw_setenv console 'ttyS0,115200'
fw_setenv autoload 'no'
fw_setenv net_check_dhcp_c 'if test $net_dhcp_c -eq 1 ; then run net_set_c_ip_dhcp; else run net_set_c_ip_stat; fi;echo ** IP Address:  $ipaddr; echo ** Subnet Mask: $netmask; echo ** Def Gateway: $gatewayip'
fw_setenv net_set_c_ip_dhcp 'dhcp; echo; echo ** Getting IP Settings by DHCP (net_dhcp_c = 1):'
fw_setenv net_set_c_ip_stat 'setenv ipaddr $net_c_ipaddr; setenv netmask $net_c_netmask; setenv gatewayip $net_c_gatewayip; echo; echo ** Using static IP Settings (net_dhcp_c = 0):'
fw_setenv net_check_dhcp_s 'if test $net_dhcp_s -eq 1 && test $net_dhcp_c -eq 1 ; then run net_set_s_ip_dhcp; else run net_set_s_ip_stat; fi; echo ** NFS Boot Server: $net_nfs_server; echo'
fw_setenv net_set_s_ip_dhcp 'setenv net_nfs_server $serverip;echo; echo ** Getting NFS Boot Server by DHCP (net_dhcp_s and net_dhcp_c = 1):'
fw_setenv net_set_s_ip_stat 'echo; echo ** Using static NFS Boot Server (net_dhcp_s or net_dhcp_c = 0) :'

fw_setenv net_load_dtb    'echo NFS loading DTB     :$net_nfs_server:$net_nfs_path/$net_nfs_dtb ...   ; nfs $load_dtb_addr    $net_nfs_server:$net_nfs_path/$net_nfs_dtb; echo'
fw_setenv net_load_initrd 'echo NFS loading uInitrd :$net_nfs_server:$net_nfs_path/$net_nfs_initrd ...; nfs $load_initrd_addr $net_nfs_server:$net_nfs_path/$net_nfs_initrd; echo'
fw_setenv net_load_uimage 'echo NFS loading uImage  :$net_nfs_server:$net_nfs_path/$net_nfs_kernel ...; nfs $load_uimage_addr $net_nfs_server:$net_nfs_path/$net_nfs_kernel; echo'

fw_setenv net_bootcmd 'run net_check_dhcp_c; run net_check_dhcp_s; run net_set_bootargs; run net_bootcmd_exec'
fw_setenv net_set_bootargs 'setenv serverip $net_nfs_server; setenv bootargs console=$console root=/dev/nfs rw rootfstype=nfs rootwait nfsroot=$net_nfs_server:$net_nfs_path/,rsize=32768,wsize=32768,hard,intr,udp,v3 ip=$ipaddr:$net_nfs_server:$gatewayip:$netmask:pogoe02:eth0:off $mtdparts $custom_params; echo ** Kernel Boot Arguments:; echo ** $bootargs; echo'
fw_setenv net_bootcmd_exec 'run net_load_uimage; if run net_load_initrd; then if run net_load_dtb; then bootm $load_uimage_addr $load_initrd_addr $load_dtb_addr; else bootm $load_uimage_addr $load_initrd_addr; fi; else if run load_dtb; then bootm $load_uimage_addr - $load_dtb_addr; else bootm $load_uimage_addr; fi; fi'
fw_setenv bootcmd 'run bootcmd_uenv; run scan_disk; run net_bootcmd; run set_bootargs; run bootcmd_exec'
```
