Lab 04 – VLAN Security and Best Practices

Objective:
Harden a basic VLAN and trunking design by applying common Layer 2 security best practices:

Move the native VLAN away from VLAN 1

Create a “parking lot” VLAN for unused ports

Disable DTP and hard-code access/trunk modes

Verify configuration with show commands

Topology

Simple 2-switch campus with a couple of PCs:

  [PC1]         [PC2]
    |             |
  Fa0/2         Fa0/2
   SW1 --------- SW2
        Fa0/24


SW1 and SW2: Cisco 2960

PC1: VLAN 10 (HR)

PC2: VLAN 20 (FINANCE)

Fa0/24: Trunk link between SW1 and SW2

Step 1. Build the Topology in Packet Tracer

Add:

2 switches (2960)

2 PCs

Cable:

PC1 → SW1 Fa0/2 (Copper Straight-Through)

PC2 → SW2 Fa0/2 (Copper Straight-Through)

SW1 Fa0/24 ↔ SW2 Fa0/24 (Copper Cross-Over)

Wait for all links to turn green.

Step 2. Create VLANs and Basic VLAN Assignments

We’ll create VLANs 10 and 20 on both switches and assign access ports.

On SW1
SW1> enable
SW1# configure terminal
SW1(config)# vlan 10
SW1(config-vlan)# name HR
SW1(config-vlan)# exit
SW1(config)# vlan 20
SW1(config-vlan)# name FINANCE
SW1(config-vlan)# exit

! PC1 in VLAN 10
SW1(config)# interface fa0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# exit

On SW2
SW2> enable
SW2# configure terminal
SW2(config)# vlan 10
SW2(config-vlan)# name HR
SW2(config-vlan)# exit
SW2(config)# vlan 20
SW2(config-vlan)# name FINANCE
SW2(config-vlan)# exit

! PC2 in VLAN 20
SW2(config)# interface fa0/2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 20
SW2(config-if)# exit

Step 3. Configure the Basic Trunk (Before Hardening)

On both switches, configure Fa0/24 as a trunk and allow VLANs 10 and 20.

! On SW1
SW1(config)# interface fa0/24
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20
SW1(config-if)# exit

! On SW2
SW2(config)# interface fa0/24
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk allowed vlan 10,20
SW2(config-if)# exit


Verify:

SW1# show interfaces trunk
SW2# show interfaces trunk


You should see Fa0/24 as a trunk with VLANs 10 and 20 allowed.

Step 4. Introduce Security VLANs (Native + Parking Lot)

Create a native VLAN (e.g., VLAN 99).

Create a parking lot VLAN for unused ports (e.g., VLAN 999).

On both switches:

SW1(config)# vlan 99
SW1(config-vlan)# name NATIVE_VLAN
SW1(config-vlan)# exit
SW1(config)# vlan 999
SW1(config-vlan)# name PARKING_LOT
SW1(config-vlan)# exit

SW2(config)# vlan 99
SW2(config-vlan)# name NATIVE_VLAN
SW2(config-vlan)# exit
SW2(config)# vlan 999
SW2(config-vlan)# name PARKING_LOT
SW2(config-vlan)# exit

Step 5. Harden the Trunk (Native VLAN + Disable DTP)

On both switches:

SW1(config)# interface fa0/24
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,99
SW1(config-if)# switchport nonegotiate
SW1(config-if)# exit

SW2(config)# interface fa0/24
SW2(config-if)# switchport trunk native vlan 99
SW2(config-if)# switchport trunk allowed vlan 10,20,99
SW2(config-if)# switchport nonegotiate
SW2(config-if)# exit


Note: switchport nonegotiate disables DTP; trunking is static.

Verify:

SW1# show interfaces fa0/24 switchport
SW1# show interfaces trunk


Check:

Mode: trunk

Native VLAN: 99

Allowed VLANs: 10,20,99

Step 6. Secure Unused Access Ports (Parking Lot VLAN)

On SW1 and SW2, choose a small range of “unused” ports (for example, Fa0/3 – Fa0/10):

SW1(config)# interface range fa0/3 - 10
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 999
SW1(config-if-range)# shutdown
SW1(config-if-range)# exit

SW2(config)# interface range fa0/3 - 10
SW2(config-if-range)# switchport mode access
SW2(config-if-range)# switchport access vlan 999
SW2(config-if-range)# shutdown
SW2(config-if-range)# exit


Verify:

SW1# show vlan brief
SW1# show interfaces status


You should see:

VLAN 999 with Fa0/3–Fa0/10 as members.

Ports showing as administratively down.

Step 7. Configure IPs on PCs and Test

On the PCs:

PC	IP Address	Subnet Mask	Gateway
PC1 (VLAN 10)	192.168.10.10	255.255.255.0	(leave blank)
PC2 (VLAN 20)	192.168.20.10	255.255.255.0	(leave blank)

There is no routing in this lab. PCs in different VLANs will not ping each other, but this is expected.

Verify:

ping within the same VLAN later if you add another host.

For now, focus on VLAN and trunk security.

Verification Checklist
Task	Command	Expected Result
VLANs exist	show vlan brief	VLAN 10, 20, 99, 999 present
Trunk native VLAN	show interfaces trunk	Native VLAN = 99 on Fa0/24
DTP disabled	show interfaces fa0/24 switchport	Administrative mode: trunk, Operational: trunk, DTP: off/nonegotiate
Unused ports secured	show vlan brief / show interfaces status	Unused ports in VLAN 999 and shut
Reflection Questions

Why is it unsafe to leave VLAN 1 as both default and native VLAN?

What is the purpose of a “parking lot” VLAN?

Why do we disable DTP and force trunk/access modes manually?

What would happen if an attacker plugged into an unused port on VLAN 1 vs VLAN 999?

Summary
Concept	Description
Native VLAN	Used for untagged traffic on a trunk; move it off VLAN 1
Parking Lot VLAN	Dedicated VLAN for unused ports, often shut
DTP Hardening	Disable dynamic negotiation, use static trunk/access
Verification	Use show vlan, show interfaces trunk, show interfaces status
