**Building Lab Environment**
Network build:
- Placed one 2911 Router, two 2960 switches, one PC and one printer
- Renamed devices to Router1, Switch1, Switch2, PC1 and Printer1 respectively
- Cabled Router1 G0/0 to Switch1 Fa0/1, Switch1 Fa0/2 to Switch2 Fa0/1, PC1 Fa0 to Switch2 Fa0/2, and Printer1 Fa0 to Switch2 Fa0/3
- Applied starter configurations on Switch1, Switch2 and Router1, establishing two VLANs (VLAN 10, 20)
- Configured PC1 and Printer1 IP settings 
	- PC1 IP address: 192.168.10.10 Default Gateway: 192.168.10.1
	- Printer1 IP address: 192.168.20.50 Default Gateway: 192.168.20.1
Verification of network connectivity:
- Successfully pinged 192.168.10.1(default gateway) from PC1
- Successfully pinged 192.168.20.50(printer) from PC1

**Access and Config Basics**
 - From the Switch2 CLI:
	 - Entered privileged EXEC mode (`enable`)
	 - Viewed and compared the running configuration with the startup configuration to verify that they are the same and that the startup configuration exists (`show running-config`) (`show startup-config`)
	 - Fa0/2 interface block was configured as:
	 `description PC1`
	 `switchport access vlan 10`
	 `switchport mode access`
	 - Fa0/3 interface block was configured as:
	 `description Printer1`
	 `switchport access vlan 20`
	 `switchport mode access`

**Ports, Interfaces & VLANs**
- Endpoint PC1 is connected to Fa0/2 on Switch2, VLAN 10
	- This connection places PC1 on the PC-side network (VLAN 10)
- Endpoint Printer1 is connected to Fa0/3 on Switch2, VLAN 20
	- This connection places Printer1 on the printer-side network (VLAN 20)
- Port Fa 0/1 is an upstream trunk port connected to Switch1, VLAN trunk
	- This connection carries traffic for both VLANs simultaneously between Switch2 and Switch1

**Finding Devices**
- Generated traffic by pinging Printer1's IP address from Router1
	- Result of ping: `Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms`
- The ping from Router1 updated the ARP table allowing Router1 to learn the MAC address of Printer1 (the first packet timed out while the device built its ARP cache)
- Ran the command `show mac address-table` from Switch2 and found the matching MAC address for Printer1, verifying the Printer1 MAC should be learned on port Fa0/3
	- VLAN: 20, MAC: 0040.0be3.a2ce, Type: DYNAMIC, Ports: Fa0/3

- *Learning the MAC address of Printer1 from Router1 allowed us to locate the printer using only the IP address, demonstrating a proper CLI workflow and detective path*

**Upstream, Neighbors and Spanning Tree** 
- Using the command `show interfaces status` on Switch2 provides information regarding the Switch2 port towards Switch1(Fa0/1), its physical connection status, and verifies it as a trunk:
	- Port: Fa0/1, Name: Uplink to SW1, Status: connected, VLAN: trunk Duplex: a-full, Speed: a-100, Type: 10/100BaseTX
- Using the command `show interfaces trunk` on Switch2 provides information regarding the VLANs being carried by the trunk
	- Port Fa0/1, Vlans allowed on trunk 10,20
	- This is an example of a healthy trunk output shape, verifying that two VLANs are being carried on the trunk between Switch1 and Switch2
- Using the command `show cdp neighbors detail` allows for use of the Cisco Discovery Protocol(CDP) to verify neighbor devices
	- Device ID: SW1, Interface: FastEthernet0/1, Port ID (outgoing port): FastEthernet0/2,  Platform: cisco 2960
 - Using the command `show spanning-tree vlan 20` provides the VLAN 20 spanning tree state
	 - Interface           Role Sts Cost      Prio.Nbr Type
		Fa0/1               Root FWD 19        128.1    P2p
		Fa0/3               Desg FWD 19        128.3    P2p
	- This shows that port Fa0/1 is forwarding vlan 20, and we know Fa0/1 is connected to Switch1 providing an uplink. This also shows port Fa0/3 is forwarding vlan 20 and we know Fa0/3 is connected to Printer1

**Health, Logs and Connectivity**
- On Switch2, command `show interfaces fastEthernet0/3` shows that the port is Up and provided information on any errors (no errors detected)
	- `956 packets input, 193351 bytes, 0 no buffer`
	`Received 956 broadcasts, 0 runts, 0 giants, 0 throttles`
	`0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored, 0 abort`
	`0 watchdog, 0 multicast, 0 pause input`
	`0 input packets with dribble condition detected`
	`2357 packets output, 263570 bytes, 0 underruns`
	`0 output errors, 0 collisions, 10 interface resets`
	`0 babbles, 0 late collision, 0 deferred`
	`0 lost carrier, 0 no carrier`
	`0 output buffer failures, 0 output buffers swapped out`
- Command `show logging` shows timeline evidence and can be paired with `show interface status` and `show running-config` to provide adequate information on the state of Switches and VLANs
	- `%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to up`
	`%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up`
	`%SYS-5-CONFIG_I: Configured from console by console`
- Further verification of connectivity provided by pinging Printer1(192.168.20.50) from Router1 with a success rate of 100 percent

**Verifying Configurations**
- On Switch2, command show running-config shows the port roles of each configured interface on the Switch:
		`interface FastEthernet0/1`
		`description Uplink to SW1`
		`switchport trunk allowed vlan 10,20`
		`switchport mode trunk`

		interface FastEthernet0/2
		description PC1
		switchport access vlan 10
		switchport mode access

		interface FastEthernet0/3
		description Printer1
		switchport access vlan 20
		switchport mode access



**Troubleshooting**
- **Scenario 1:** A user cannot print to Printer1. The ticket says its expected address is 192.168.20.50, its expected access port is SW2 Fa0/3, and printers belong in VLAN 20. “Offline” is only the reported symptom.
- Can the printer answer from its gateway?
	- Router1 ping 192.168.20.50: successful
- Did Router1 learn a MAC address for Printer1?
	- Yes, `show arp` gives an entry for IP address 192.168.20.50
	- MAC address: 0040.0be3.a2ce
- Which Switch2 port learned that MAC address?
	- MAC address 0040.0be3.a2ce was learned on port Fa0/3 at VLAN 20 (expected result
- Is port Fa0/3 physically usable?
	- Yes, `show interfaces status` and `show interfaces fastEthernet0/3` verifies that Fa0/3 is connected, in the correct VLAN, and up with zero errors
- Is Fa0/3 actually configured for the printer network?
	- Yes, `show running config` confirms the intended switchport mode (access) and access VLAN (20)
	- `show vlan brief` confirms that VLAN 20 exists and Fa0/3 belongs to it
- Did the port change state around the time of the ticket?
	- `show logging` provides timing and history. In this scenario, the log could show repeated flaps indicating a possible cabling or hardware issue. It is also possible that there is no relevant log entry meaning the switch did not record any useful events.

- **Scenario 2:** The endpoint's(Printer1) link light is on, but its IP address is in the wrong subnet or it cannot reach the gateway for its expected VLAN. Physical connectivity exists, so test VLAN placement before blaming the uplink.
- Which port is the endpoint using, and which VLAN does the switch report?
	- `show interfaces status` verifies that Fa0/3 is the port Printer1 is connected to and that it is in the correct VLAN (20)
- Does the saved intent match the operational summary?
	- `show running-config` and `show vlan brief`  shows that Fa0/3 is configured for VLAN 20 and appears under VLAN 20 confirming that the local access-port configuration is internally consistent
- If the access port is correct, can that VLAN leave Switch2?
	- `show interfaces trunk` provides the information that Fa0/1 on Switch2 is trunking VLANs 10 and 20
	- In this scenario, if the infrastructure looks correct, and all Switch ports and VLANs are properly configured, record the evidence and escalate the ticket. If the VLANs are not properly configured, then an escalation or change request would be the proper next step 

- **Scenario 3:** Multiple endpoints in VLAN 20 fail at the same time, or local ports look correct but none of those endpoints can reach their gateway. A shared failure makes the shared uplink or upstream device more likely than several simultaneous endpoint failures.
- Is the shared trunk operational and carrying the affected VLAN
	- Yes, `show interfaces trunk` confirms that Fa0/1 is trunking and VLAN 20 is allowed
- Does Fa0/1 lead to the device you think it does?
	-  Yes, running the command `show cdp neighbors detail` identifies Switch1 on Switch2 Fa0/1
- Can the gateway reach the endpoint network?
	- Yes, pinging Printer1 (192.168.20.50) from Router1 is successful and confirms reachability; `traceroute` confirms zero failures.
	- Given this information, all configurations appear to be correct and as a Tier 1 Help Desk professional, this ticket would be escalated with all outputs attached.

All three troubleshooting scenarios demonstrate proper troubleshooting and workflow by picking commands based on evidence, producing short ticket notes with commands used and what those commands prove, and knowing when to escalate a ticket, not experiment on an active network, potentially causing more user issues. Using the current network configuration, all three scenarios point to escalating the user tickets. From a logical standpoint, VLAN 20 is working correctly within this network configuration. It is likely the cause is a Layer 1 issue on Printer1 itself.
