**Devices:**  
- 2911 Router (Router1) 
- 2960 Switch (Switch 1)
- 2960 Switch (Switch 2)
- PC (PC1)
- Printer (Printer 1)

**Cable Topology:**
Router1 GigabitEthernet0/0 -> Switch1 FastEthernet0/1
Switch1 FastEthernet0/2  -> Switch2 FastEthernet0/1
PC1 FastEthernet0    -> Switch2 FastEthernet0/2
Printer1 FastEthernet0 -> Switch2 FastEthernet0/3

**Switch1 CLI config:**

`Switch>enable`
`Switch#configure terminal`
`Enter configuration commands, one per line. End with CNTL/Z.`
`Switch(config)#hostname Switch1`
`Switch1(config)#interface fastEthernet0/1`
`Switch1(config-if)#description uplink to Router1`
`Switch1(config-if)#switchport trunk allowed vlan 10,20`
`Switch1(config-if)#switchport mode trunk`
`Switch1(config-if)#interface fastEthernet0/2`
`Switch1(config-if)#description downstream to Switch2`
`Switch1(config-if)#switchport trunk allowed vlan 10,20`
`Switch1(config-if)#switchport mode trunk`
`Switch1(config-if)#`
`%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down`

  `%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up`

`end`

`Switch1#`
`%SYS-5-CONFIG_I: Configured from console by console`
`copy running-config startup-config`
`Destination filename [startup-config]?`
`Building configuration...`

`[OK]`

**Switch2 CLI config:**

`Switch>enable`
`Switch#configure terminal`
`Enter configuration commands, one per line. End with CNTL/Z.`
`Switch(config)#hostname Switch2`
`Switch2(config)#interface fastEthernet0/1`
`Switch2(config-if)#description Uplink to Switch1`
`Switch2(config-if)#switchport trunk allowed vlan 10,20`
`Switch2(config-if)#switchport mode trunk`
`Switch2(config-if)#interface fastEthernet0/2`
`Switch2(config-if)#description PC1`
`Switch2(config-if)#switchport access vlan 10`
`% Access VLAN does not exist. Creating vlan 10`
`Switch2(config-if)#switchport mode access`
`Switch2(config-if)#interface fastEthernet0/3`
`Switch2(config-if)#description Printer1`
`Switch2(config-if)#switchport access vlan 20`
`% Access VLAN does not exist. Creating vlan 20`
`Switch2(config-if)#switchport mode access`
`Switch2(config-if)#end`
`Switch2#`
`%SYS-5-CONFIG_I: Configured from console by console`
`copy running-config startup-config`
`Destination filename [startup-config]?`
`Building configuration...`
`[OK]`

**Router1 CLI config:**

`Router>enable`
`Router#configure terminal`
`Enter configuration commands, one per line. End with CNTL/Z.`
`Router(config)#hostname Router1`
`Router1(config)#interface gigabitEthernet0/0`
`Router1(config-if)#description Trunk to Switch1 G0/1`
`Router1(config-if)#no shutdown`
`Router1(config-if)#`

`%LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to up`

`%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up`

`Router1(config-if)#interface gigabitEthernet0/0.10`
`Router1(config-subif)#`

`%LINK-3-UPDOWN: Interface GigabitEthernet0/0.10, changed state to down`

`%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0.10, changed state to up`

  `Router1(config-subif)#encapsulation dot1Q 10`
`Router1(config-subif)#ip address 192.168.10.1 255.255.255.0`
`Router1(config-subif)#interface gigabitEthernet0/0.20`
`Router1(config-subif)#`

`%LINK-3-UPDOWN: Interface GigabitEthernet0/0.20, changed state to down`

`%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0.20, changed state to up`

  `Router1(config-subif)#encapsulation dot1Q 20`
`Router1(config-subif)#ip address 192.168.20.1 255.255.255.0`
`Router1(config-subif)#end`

`Router1#`

`%SYS-5-CONFIG_I: Configured from console by console`

`Router1#copy running-config startup-config`
`Destination filename [startup-config]?`
`Building configuration...`
`[OK]`
