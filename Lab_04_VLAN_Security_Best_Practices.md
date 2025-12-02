# Lab 04 – VLAN Security and Best Practices

**Objective:**  
Harden a basic VLAN and trunking design by applying common Layer 2 security best practices:

- Move the native VLAN away from VLAN 1  
- Create a “parking lot” VLAN for unused ports  
- Disable DTP and statically configure trunk/access modes  
- Verify configuration with *show* commands  

---

## Topology

```
  [PC1]         [PC2]
    |             |
  Fa0/2         Fa0/2
   SW1 --------- SW2
        Fa0/24
```

- **SW1 and SW2:** Cisco 2960  
- **PC1:** VLAN 10 (HR)  
- **PC2:** VLAN 20 (FINANCE)  
- **Fa0/24:** Trunk link between SW1 and SW2  

---

## Step 1. Build the Topology in Packet Tracer

1. Add **2 switches (2960)**  
2. Add **2 PCs**  
3. Cable connections:  
   - PC1 → SW1 Fa0/2 (**Straight-Through**)  
   - PC2 → SW2 Fa0/2 (**Straight-Through**)  
   - SW1 Fa0/24 ↔ SW2 Fa0/24 (**Cross-Over**)  

Wait until all links show **green/up**.

---

## Step 2. Create VLANs and Basic VLAN Assignments

### On SW1
```
SW1> enable
SW1# configure terminal
SW1(config)# vlan 10
SW1(config-vlan)# name HR
SW1(config)# vlan 20
SW1(config-vlan)# name FINANCE

! PC1 in VLAN 10
SW1(config)# interface fa0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
```

### On SW2
```
SW2> enable
SW2# configure terminal
SW2(config)# vlan 10
SW2(config-vlan)# name HR
SW2(config)# vlan 20
SW2(config-vlan)# name FINANCE

! PC2 in VLAN 20
SW2(config)# interface fa0/2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 20
```

---

## Step 3. Configure the Basic Trunk (Before Security Hardening)

### On SW1
```
SW1(config)# interface fa0/24
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20
```

### On SW2
```
SW2(config)# interface fa0/24
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk allowed vlan 10,20
```

### Verify
```
show interfaces trunk
```

Expected: Fa0/24 listed as a trunk with VLANs **10,20** allowed.

---

## Step 4. Introduce Security VLANs (Native VLAN + Parking Lot VLAN)

### On both switches
```
vlan 99
 name NATIVE_VLAN
vlan 999
 name PARKING_LOT
```

---

## Step 5. Harden the Trunk (Native VLAN + Disable DTP)

### On both switches
```
interface fa0/24
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 switchport nonegotiate
```

**Note:** `switchport nonegotiate` disables DTP; trunking is now **static**.

### Verify
```
show interfaces fa0/24 switchport
show interfaces trunk
```

**Expected:**

- Mode: trunk  
- Native VLAN: **99**  
- Allowed VLANs: **10,20,99**  
- DTP: **off / nonegotiate**  

---

## Step 6. Secure Unused Ports (Parking Lot VLAN)

Example unused ports: Fa0/3–Fa0/10

### On SW1 and SW2
```
interface range fa0/3 - 10
 switchport mode access
 switchport access vlan 999
 shutdown
```

### Verify
```
show vlan brief
show interfaces status
```

Expected:

- Ports in VLAN **999**  
- Ports **administratively down**

---

## Step 7. Configure PC IP Addresses and Test

| PC  | VLAN | IP Address       | Subnet Mask     | Gateway |
|-----|------|------------------|-----------------|---------|
| PC1 | 10   | 192.168.10.10    | 255.255.255.0   | (blank) |
| PC2 | 20   | 192.168.20.10    | 255.255.255.0   | (blank) |

There is **no routing** in this lab.  
PCs in different VLANs will **not** ping each other (expected).

---

## Verification Checklist

| Task                 | Command                             | Expected Result                          |
|----------------------|--------------------------------------|------------------------------------------|
| VLANs exist          | `show vlan brief`                   | VLANs 10, 20, 99, 999 present            |
| Native VLAN set      | `show interfaces trunk`             | Native VLAN = 99                         |
| DTP disabled         | `show interfaces fa0/24 switchport` | DTP: off / nonegotiate                   |
| Unused ports secured | `show interfaces status`            | Fa0/3–10 in VLAN 999 + shutdown          |

---

## Reflection Questions

- Why is it unsafe to leave VLAN 1 as the default and native VLAN?  
- What is the purpose of a parking lot VLAN?  
- Why should DTP be disabled on trunk/access ports?  
- What happens if an attacker plugs into:  
  - **VLAN 1** vs VLAN 999** ?  

---

## Summary

| Concept           | Description                                                 |
|-------------------|-------------------------------------------------------------|
| **Native VLAN**   | Used for untagged traffic; should not be VLAN 1            |
| **Parking Lot VLAN** | For unused ports; typically shut down                   |
| **DTP Hardening** | Forces static trunk/access modes; prevents rogue trunking   |
| **Verification**  | Use `show vlan`, `show interfaces trunk`, `show interfaces status` |

```
