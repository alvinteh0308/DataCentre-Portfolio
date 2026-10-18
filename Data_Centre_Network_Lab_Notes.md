**Configure SVI in Core SW**
```bash
en
conf t
vlan 10
name APPLICATION
exit
int vlan 10
ip add 192.168.10.1 255.255.255.0
no shut
end
sh ip int br
```

**Configure VLAN and assign port in access SW**
```bash
configure terminal
vlan 10
name APPLICATION
exit
interface fastEthernet 0/1
switchport mode access
switchport access vlan 10
exit
end
```

**configure trunk link to access SW**
```bash
en
conf t
int gi0/1
sw mo trunk
sw trunk allowed vlan 10,20,30,40
end
sh int trunk
```

**Configure trunk link to Core SW**
```bash
en
conf t
int gi0/1
sw trunk encapsulation dot1q
sw mo trunk
sw trunk allowed vlan 10,20,30,40
end
```

**enable ip routing in Core SW**
```bash
en
conf t
ip routing
end
sh ip route
```

##ACL (Access Control List)
**Create ACL in core SW, not enable yet**
```bash
en
conf t
ip access-list extended GUEST-ISOLATION
deny ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
deny ip 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
permit ip any any
exit
end
sh access-lists
```

**enable ACL in interface**
```bash
en
conf t
int vlan 40
ip access-group GUEST-ISOLATION in
end
```

## Step of setup SSh remote access
**check ssh status in Core SW**
```bash
en
sh ip ssh
```

**configure hostname and domain name in Core SW**
this is use for generate RSA Key Pair, for SSH handshack and verification purpose
```bash
en
conf t
hostname CORE-SW1
ip domain-name datacentre.lab
end
```

**Create Local Username and PSW in core sw**
```bash
en
conf t
username dcadmin privilege 15 secret DcLab2026!
end
```

**generate RSAkey in core SW**
when choose Key size, choose biggest one, more secure
```bash
en
conf t
crypto key generate rsa
end
sh ip ssh
```

**configure as ssh version2**
```bash
conf t
ip ssh version 2
end
sh ip ssh
```

**configure VTY(Virtual Teletype) on Core SW**
```bash
en
conf t
line vty 0 4
login local
transport input ssh
exit
end
```

**ssh remote from pc**
```bash
ssh -l dcadmin gateway
```

**ACL for SSH management**
```bash
en
conf t
ip access-list standard SSH-MANAGEMENT
permit host 192.168.20.10
deny any
exit
end
sh access-lists
```

**apply access-class in line VTY**
```bash
en
conf t
line vty 0 4
access-class SSH-MANAGEMENT in
end
```

## DHCP assign
**exclude ip assign to devices**
```bash
en
conf t
ip dhcp excluded-address 192.168.40.1 192.168.40.20
end
```

**create DHCP pool**
```bash
en
conf t
ip dhcp pool GUEST-NETWORK
network 192.168.40.0 255.255.255.0
default-router 192.168.40.1
dns-server 192.168.10.20
end
```

##Create DNS Server
open dns server -> services -> DNS -> add name, domain (app.datacentre.lab), ip
set dns ip address on client device(admin pc)

##Network Redundancy
**configure STP root bridge on Core SW1**
```bash
en
conf t
spanning-tree vlan 10,20,30,40 priority 4096
end
```

**configure second root bridge on Core SW2**
```bash
en
conf t
spanning-tree vlan 10,20,30,40 priority 8192
end
```

##configure HSRP
**configure SVI VLAN 20 on SW2**
```bash
en
conf t
ip routing
int vlan 20
ip address 192.168.20.3 255.255.255.0
no shut
end
```

**configure core2core trunk SW1**
```bash
en
conf t
int fa0/24
sw tr encapsulation dot1q
sw mo trunk
sw tr allowed vlan 10,20,30,40
no shut
end
```

**configure core2core trunk SW2**
```bash
en
conf t
int fa0/24
sw tr encapsulation dot1q
sw mo trunk
sw tr allowed vlan 10,20,30,40
no shut
end
```

**Configure HSRP on CORE-SW2**
```bash
en
conf t
int vlan 20
standby 20 priority 100
standby 20 preempt
end
```

**Configure SW1 new ip and set in HSRP group**
```bash
en
conf t
int vlan 20
ip address 192.168.20.2 255.255.255.0
standby 20 ip 192.168.20.1
standby 20 priority 110
standby 20 preempt
end
```

**Configure SW2 set in HSRP group**
```bash
en
conf t
int vlan 20
standby 20 ip 192.168.20.1
end
```


##configure SW1 to record syslog to server
```bash
en
conf t
logging 192.168.10.20
logging trap informational
end
```



show interfaces status
show int gi0/1
show logging
show ip interface brief （显示所有layer3 ip add状态）
show vlan brief
copy running-config startup-config
sh mac address-table
sh int trunk
sh ip dhcp pool
sh ip dhcp binding
sh spanning-tree vlan 10
sh standby br
