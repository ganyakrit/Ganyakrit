---
layout: post
published: true
title: ICND1 Lab Compilation
---
## ICND1


RESET HW

Reset HW configuration and account

connect console to computer and press Break button on starting SW , after SW boot to rommon 

confreg  0x2142 

delete flash : vlan.dat	

delete flash: config.txt 

erase startup-config 

reload 

erase nvram 

erase st 

reload 

Initial SW Config 

this step to config how to connect the switch with CON, VTY (telnet , SSH) by config the user account and password for each connection and Enable Exec mode. Let start with connect PC to SW with console 

to set hostname on Switch 

Switch> enable 

Switch(config)# config terminal 

Switch(config)# hostname SW

Switch(config)# no ip source-route

Switch(config)# no ip domain-lookup

1. Enable Privilege Exec 

Enable has 2 type -- password and secret. You should use encryption for password

SW(config)# enable password password

SW(config)# service password-encryption

or to use secret instead (secret use MD5 encryption)

SW(config)# enable secret  *secret*

Switch will use secret if both was set

2. Line Console

SW#config **terminal** 

SW(config)#line **console ****_0_**** **

SW(config-line)# **password ****_consolepassword_**

SW(config-line)# **login**

3. VTY  telnet 

assume this is initial config I will connect vlan1 ip address on SW and connect to the PC with 192.168.1.1 

SW(config)# **interface vlan ****_1_**** **

SW(config-if)# **ip address ****_192.168.1.2_**** ****_255.255.255.0_**

SW(config-if)# **no shut**

SW(config-if)# **exit**

SW(config)# **ip default-gateway 192.168.1.1**

from now on I will connect LAN cable to the any port of SW for telnet to the SW  but cannot connected 

SW# c**onfig terminal**

SW(config)# **line vty 0 4 **

SW(config-line)# **password ****_telnetpassword_**

SW(config-line)# **exec-timeout 15**

SW(config-line)# **login** 

SW(config-line)# **end**

C:\>telnet 192.168.1.2

Trying 192.168.1.2 ...Open

User Access Verification

Password: 

4. VTY SSH

To use SSH connect to config domain name , crypto key , username and password 

SW(config)# **ip domain-name ****_sky.net_**

SW(config)# **crypto key generate rsa**

The name for the keys will be: SW.sky.net

Choose the size of the key modulus in the range of 360 to 2048 for your

General Purpose Keys. Choosing a key modulus greater than 512 may take

a few minutes.

How many bits in the modulus [512]: **_1024_**

% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]

SW(config)# u**sername ****_cisco_**** secret ****_cisco123_****	**

SW(config)# **line vty 0 15**

SW(config-line)# **login local**

SW(config-line)# **no password**

SW(config-line)#

SW(config-line)# **transport input ssh**

SW(config-line)# **ip ssh version 2**

Please note, connection from SSH may use  password from vty 0 4 config if inplace, so I suggest to use no password and login local for vty 0 4 and vty 5  15 . then when you login with telnet, issue username and password 

 

STIG Suggest to first setup hardening 

1. Disable Vlan 1 

2. Create vlan for disable Vlan 

3. Assign all port to disable Vlan 

4. Assign all port to access mode 

5. Disable all port 

6. Create dedicated Management Vlan 

7. Change native Vlan to 488 (on trunk port) 

8. Config manually each port you use !!!

9. Disable cdp and http 

SW(config)# **interface vlan 1**

SW(config-if)# **no ip address**

SW(config-if)# **shut**

SW(config)# **vlan 666**

SW(config-vlan)# **name DISABLE-VLAN**

	

SW(config)# **interface range fastEthernet 0/1-24**

SW(config-if-range)# s**witchport mode access**

SW(config-if-range)# s**witchport access vlan 666**

SW(config-if-range)# s**hut**

SW(config)# **vlan 999**

SW(config-vlan)# **name MANAGEMENT-VLAN**

/* set trunk interface */

SW(config)# **interface gigabitEthernet 0/1**

SW(config-if)# **switchport trunk**

SW(config-if)# **switchport trunk native vlan 488**

?? does it need to config native vlan on L3 Sw or Router interface : YES

L2 and L3 SW trunking 

To set L2 SW to trunk 

SW#config **terminal** 

SW(config)# **interface gi0/2**

SW(config-if)# **switchport mode trunk**

SW(config-if)# **switchport trunk native vlan 488**

SW(config-if)# **switchport nonegotiate**

To set L3 SW interface to trunk

SW(config)# **vlan 10** 

SW(config-if)# **name vlan-name**

MLS(config)# **config terminal **

MLS(config)# **interface gi0/2**

MLS(config-if)# **switchport trunk encapsulation dot1q**

MLS(config-if)# **switchport mode trunk**

MLS(config-if)# **switchport trunk native vlan 488**

MLS(config-if)# **end **

Please note if you not config interface L3SW as trunk you will not be able to set dot1q and you will receive CDP mismatch native Vlan .. 

SW1(config)#int range gi0/1 -2

SW1(config-if-range)#channel-group 1 mode desirable

SW1(config-if-range)#exit

SW1(config)#interface port-channel  1

SW1(config-if)#switchport trunk encapsulation dot1q

SW1(config-if)#switchport trunk native vlan 488

SW1(config-if)#switchport mode trunk 

SW2(config)#int range gi0/1 -2

SW2(config-if-range)#channel-group 1 mode desirable

SW2(config-if-range)#exit

SW2(config)#interface port-channel  1

SW2(config-if)#switchport trunk encapsulation dot1q

SW2(config-if)#switchport trunk native vlan 488

SW2(config-if)#switchport mode trunk 

To Set SW to manage Vlan Database 

MLS#conf **t**

MLS(config)# **vtp domain vpt.sky.net**

MLS(config)# **vtp mode ****[server | client  | transparent]**

DIAGNOSTIC SW and TRUNKING 

SW#sh run 

SW#sh run interface *int* /*only work with real equipment

SW#sh interface [int] status /*summary int -- vlan 

SW#sh interface int [trunk | switchport] /* L1 detail

SW#sh interfaces [trunk | switchport ]

SW#sh interfaces 

SW(config-if)#switchport trunk allowed vlan [add | remove] list

SW#sh ip interface brief

SW#sh ip arp

SW#sh ip ssh

SW#sh vlan 

SW#sh vtp counter 

SW#sh vtp status 

SW# sh mac address-table 

SW# sh mac address-table dynamic 

DHCP by L3SW or Router

MLS(config)# **ip dhcp pool DHCP_POOL**

MLS(dhcp-config)# **network 192.168.10.0 255.255.255.0**

MLS(dhcp-config)# **default-router 192.168.10.1**

MLS(dhcp-config)# **dns-server 192.168.10.1**

MLS(dhcp-config)# **exit**

MLS(config)# **ip dhcp excluded-address 192.168.10.1**

MLS(config)# **end**

MLS# **show ip dhcp binding** 

MLS# **show ip dhcp server statistics**

**MLS# show ip dhcp **

**MLS# show ip dhcp lease**

**MLS# show ip dhcp pool **

**MLS# debug ip dhcp server packet **

To use dhcp on computer or router interface 

# ip address dhcp 

# ip dhcp client *client-id *ascii *name *

### DHCP relay

Notice that the command is entered on the interface that will receive DHCPv4 broadcasts. R1 then forwards DHCPv4 broadcast messages as a unicast to 192.168.11.5.

 By  default, the ip helper-address command forwards the following eight UDP services:

Port 37: Time

Port 49: TACACS

Port 53: DNS

Port 67: DHCP/BOOTP client

Port 68: DHCP/BOOTP server

Port 69: TFTP

Port 137: NetBIOS name service

Port 138: NetBIOS datagram service

R1(config)# **interface gigabitethernet 0/0**

R1(config-if)# **ip helper-address 192.168.11.5**

To specify additional ports, use the global command 

**ip forward-protocol udp [port-number | protocol]**

To disable broadcasts of a particular protocol, use the no form of the command.

Remark  if you use sub-interface , you should be issue ip helper on sub-interface also  ?  

Inter VLAN Routing 

In ccna R&S we use 3 type of Vlan routing

#### 1. L3 Routing

##### 1.1 L2 Link

In this method imaging that your can connection many trunk port from SW to L3SW interface, that mean may have many Vlan on a trunk. All you have to do after create each vlan on the L3SW are creating Interface VLAN, assign IP address and issue command to route 

MLS(config)# **vlan 10**

MLS(config)# **vlan 30**

MLS(config)# **interface vlan 10**

MLS(config-if)# **ip address 172.17.10.1 255.255.255.0**

MLS(config-if)# **interface vlan 30**

MLS(config-if)# **ip address 172.30.10.1 255.255.255.0**

MLS(config-if)# **interface f0/11**

MLS(config-if)# **switchport mode access**

MLS(config-if)# **switchport access vlan 10**

MLS(config-if)# **interface f0/6**

MLS(config-if)# **switchport mode access**

MLS(config-if)# **switchport access vlan 30**

MLS(config-if)# **end**

Do these command each vlan you have, and set routing 

MLS(config)# **ip routing** 

##### 1.2 L3 Link

this method set interface to perform like router interface, not a switchport, set ip address and routing 

MLS(config)# **interface g0/1**

MLS(config-if)# **no switchport**

MLS(config-if)# **ip address 192.0.2.2 255.255.255.0**

!The **no shutdown** command is not required because switch interfaces are already activated.

MLS(config-if)# **exit**

MLS(config)# **ip route 0.0.0.0 0.0.0.0 g0/1**

MLS(config)# **exit**

MLS# **show ip route**

 

#### 2 Router on a STICK ( sub interface )

![image alt text](image_0.png)

Please note: sh vlans (only on equib)

This method use one interface from router to route all Vlan on the trunk connected to interface, have to define encapsulation on Router interface matched with Vlan Number for each vlan 

R1(config)# **interface gi0/0**

R1(config-if)# **no shutdown**

R1(config-if)# **interface g0/0.10**

R1(config-subif)# **encapsulation dot1q 10**

R1(config-subif)# **ip address ****192****.****168****.10.1 255.255.255.0**

R1(config-subif)# **interface g0/****0****.****2****0**

R1(config-subif)# **encapsulation dot1q ****2****0**

R1(config-subif)# **ip address ****192****.****168****.****2****0.1 255.255.255.0**

SW1(config)#int gi0/0

SW1(config-if)#switchport trunk encapsulation dot1q

SW1(config-if)#switchport mode trunk

SW1(config-if)#switchport trunk native vlan 488

SW2(config)#int gi3/3

SW2(config-if)#switchport mode access

SW2(config-if)#switchport access vlan 10

SW2(config-if)#

SW3(config)#int gi3/3

SW3(config-if)#switchport mode access

SW3(config-if)#switchport access vlan 20

SW3(config-if)#

#### 3.Router Interface ( one interface for one network ) 

![image alt text](image_1.png)

This type of vlan routing will have router interface as subnet number 

Each vlan connect direct to the Router interface with Access mode 

All Switch has Vlan10 , Vlan20 , Vlan 999 , native vlan is vlan488

Link from Sw1 to R1 is Access Mode of it’s Vlan (except vlan 999 in this lab) 

Link from Switch to Switch is trunk Link

Link From SW1 to Sw2 is Port-channel 

pc-1 using Vlan10 , pc-2 using Vlan20

R1(config)#int gi1/0

R1(config-if)#ip address 192.168.10.1 255.255.255.0

R1(config-if)#no shut

R1(config-if)#int gi0/0

R1(config-if)#ip address 192.168.20.1 255.255.255.0

R1(config-if)#no shut

SW1(config)#int gi 1/0

SW1(config-if)#sw

SW1(config-if)#switchport mode access

SW1(config-if)#switchport access vlan 10

SW1(config-if)#int gi 0/0

SW1(config-if)#switchport mode access

SW1(config-if)#switchport access vlan 20

SW1(config-if)#exit

SW1(config)#

SW2(config)#int gi3/3

SW2(config-if)#switchport mode access

SW2(config-if)#switchport access vlan 10

SW2(config-if)#

SW3(config)#int gi3/3

SW3(config-if)#switchport mode access

SW3(config-if)#switchport access vlan 20

SW3(config-if)#

. 

Router Inspection 

Show ip interface brief

Show protocols

Show interfaces

Show interface int

Show ip route

Show run interface int 

Show ip arp 

# ROUTING 

STATIC ROUTE 

R1 # config terminal 

R1(config)# ip route **_netid_**** ***subnetmask *[nexthop | int ]

Default Route 

R1(config)# config terminal 

R1(config)# ip route **_netid_**** ***subnetmask *[nexthop | int ]

Verify : 
Show ip route static 

Show running-config  |  include route 

RIP v2 

# router rip 

Version 2 

Network *network-id *

No auto-summary

Clear ip route 

Passive-interface [int  | default ]

Default-information originate 

Ip route advertise *time *

Timer basic 60 180 180 245 

Port Security 

#switchport port-security 

#switchport port-security maximum n

#switchport port-security mac-address []

#switchport violation [shutdown | protect | restrict | protect ]

Verify 

# show port-security 

#show port-security interface int 

#show interface status  /* see err-disabled 

#show port-security address /* show mac - interface 

Set to recovery by auto 

Errdisable recovery cause psecure-violation 

Errordisable recovery interval 30 

Show errdiable recovery 

# ACL 

Set access control to Switch or Router 

# access-list *accnumber * [permit | deny ] network wildcard

Assign access list to port  

1. Line vty 

#line vty  n n 

# access-class accnumber [in | out ]

2. Interface

# interface int 

# ip access-group accnumber [in | out ]

# NAT 

1 Static Nat 

Static nat is one-to-one translate 

# ip nat static source static *inside-ip * *outside-ip* 

2. Dynamic Nat

Dynamic nat is Many-to-Many translate 

This translate have to create access-list for the IP-group or network to be use 

# access-list accnumber [permit | deny ] *network **wildcard*

Create ip pool ip  (outside IP ) 

Ip nat *pool-name ip * *ip netmask mask * 

Then assign access list to the use on nat 

# ip nat inside source list *accnum* *pool-name *

3. PAT 

Pat is may-to-one translate, it use one ip (usually interface ip ) to be global address , this have to create access-list  and assigned to nat 

# ip nat inside source list *accnum interface  *overload 

Verify

Show control-plane host open-ports

Show ip nat translations 

Show ip nat statistics 

Show access-list *accnum*

Debug ip nat 

Clear ip nat translate 

# Device Management

Logging to Syslog 

SW1(config)# logging trap informational

SW1(config)# logging host 10.3.10.50

SW1(config)# show logging 

NTP 

SW1(config)# ntp server ip-addr

SW1(config)# ntp master statum 

SW1(config)# show ntp associations

SW1(config)# show ntp status 

SW1(config)# clock timezone CET 2 

Radius authentication

SW1(config)# radius server QNAP-RADIUS

SW1(config-radius-server)# address ipv4 10.3.10.50

SW1(config-radius-server)# key radiuskey

SW1(config-radius-server)# exit

SW1(config)# aaa new-model

SW1(config)# aaa authentication login default radius local 

802.1x 

SW1(config)#aaa authentication dot1x default group radius

SW1(config)#dot1x system-auth-control

SW1(config-if)#dot1x port-control auto

SW1(config-if)# dot1x pae authenticator

802.1x Config at EndPoint Computer 

 

BASIC ASA FireWall With GNS3 

Please refer to GNS3 installation and config to run ASAv and IOSv on GNS3 

To Initial ASA with 3 Zone 

![image alt text](image_2.png)

1. Interface and IP Assignment 

2. Config route to internet 

3. Config NAT From Inside 

4. Config Inspect Policy

5. Config DMZ Static NAT 

6. Config DMZ ACL
