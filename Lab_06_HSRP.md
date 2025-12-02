# Lab 06 – First-Hop Redundancy with HSRP

**Objective:**  
Configure HSRP to provide a redundant default gateway for hosts in a VLAN.

---

## Topology

Single VLAN with two routers acting as redundant gateways.

```
          (Trunk)
       Fa0/1     Fa0/1
      [R1]-------[SW1]-------[PC1]
       |         Fa0/2
       |       
      [R2]
      Fa0/1 (Trunk)
```

R1 and R2: Routers using router-on-a-stick for VLAN 10  
SW1: Access switch  
PC1: Host in VLAN 10 using HSRP virtual IP as default gateway

---

## IP Plan (VLAN 10 – 192.168.10.0/24)

| Device | Interface | IP Address |
|--------|-----------|-------------|
| R1 | G0/0.10 | 192.168.10.2/24 |
| R2 | G0/0.10 | 192.168.10.3/24 |
| HSRP Virtual | — | 192.168.10.1/24 |
| PC1 | NIC | 192.168.10.10/24, GW 192.168.10.1 |

R1 will be **Active**, R2 will be **Standby**.

---

## Step 1. Build the Topology

Add:

- 2 routers (e.g., 2911)  
- 1 switch (SW1, 2960)  
- 1 PC  

Cabling:

- R1 G0/0 → SW1 Fa0/1  
- R2 G0/0 → SW1 Fa0/3  
- PC1 → SW1 Fa0/2  

---

## Step 2. Configure VLAN 10 on SW1

```
SW1> enable
SW1# configure terminal
SW1(config)# vlan 10
SW1(config-vlan)# name USERS
SW1(config-vlan)# exit
```

Assign ports:

### PC1 Access Port
```
SW1(config)# interface fa0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# exit
```

### Trunk to R1
```
SW1(config)# interface fa0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10
SW1(config-if)# exit
```

### Trunk to R2
```
SW1(config)# interface fa0/3
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10
SW1(config-if)# exit
```

---

## Step 3. Configure Router Subinterfaces for VLAN 10

### On R1
```
R1> enable
R1# configure terminal
R1(config)# interface gigabitEthernet0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.2 255.255.255.0
R1(config-subif)# no shutdown
R1(config-subif)# exit
R1(config)# interface gigabitEthernet0/0
R1(config-if)# no shutdown
R1(config-if)# exit
```

### On R2
```
R2> enable
R2# configure terminal
R2(config)# interface gigabitEthernet0/0.10
R2(config-subif)# encapsulation dot1Q 10
R2(config-subif)# ip address 192.168.10.3 255.255.255.0
R2(config-subif)# no shutdown
R2(config-subif)# exit
R2(config)# interface gigabitEthernet0/0
R2(config-if)# no shutdown
R2(config-if)# exit
```

---

## Step 4. Configure HSRP on VLAN 10

We’ll use group **10** and virtual IP **192.168.10.1**.

### On R1 (Active)
```
R1(config)# interface gigabitEthernet0/0.10
R1(config-subif)# standby 10 ip 192.168.10.1
R1(config-subif)# standby 10 priority 110
R1(config-subif)# standby 10 preempt
R1(config-subif)# standby 10 description HSRP_GROUP_10_ACTIVE
```

### On R2 (Standby)
```
R2(config)# interface gigabitEthernet0/0.10
R2(config-subif)# standby 10 ip 192.168.10.1
R2(config-subif)# standby 10 priority 100
R2(config-subif)# standby 10 preempt
R2(config-subif)# standby 10 description HSRP_GROUP_10_STANDBY
```

---

## Step 5. Configure PC1

- IP: 192.168.10.10  
- Mask: 255.255.255.0  
- Default Gateway: **192.168.10.1**  

---

## Step 6. Test Connectivity and HSRP Failover

### 1. Basic Connectivity

From PC1:
```
ping 192.168.10.1
ping 192.168.10.2
ping 192.168.10.3
```

### 2. HSRP Status

Check on both routers:
```
show standby brief
```

### Expected:
- R1 → Active  
- R2 → Standby  

### 3. Simulate Failover

Shut R1’s subinterface:
```
R1(config)# interface gigabitEthernet0/0.10
R1(config-if)# shutdown
```

R2 should take over. Pings to 192.168.10.1 should continue.

### 4. Restore R1 and Watch Preemption
```
R1(config)# interface gigabitEthernet0/0.10
R1(config-if)# no shutdown
```

R1 should become Active again.

---

## Verification Checklist

| Task | Command | Expected Result |
|------|---------|----------------|
| HSRP group formed | show standby brief | Group 10, VIP 192.168.10.1 |
| R1 active | show standby brief (R1) | State = Active |
| R2 standby | show standby brief (R2) | State = Standby |
| PC gateway | ping 192.168.10.1 | Success |
| Failover | shutdown R1 G0/0.10 | R2 becomes Active |

---

## Reflection Questions

- Why does PC1 point to the virtual IP instead of R1 or R2?  
- What is the benefit of preempt?  
- What happens if both routers have the same priority?  
- How would you safely test failover in production?

---

## Summary

| Concept | Description |
|---------|-------------|
| HSRP | First-hop redundancy protocol |
| Virtual IP/MAC | Shared gateway for hosts |
| Active/Standby roles | Active forwards traffic |
| Failover | automatic gateway recovery |

