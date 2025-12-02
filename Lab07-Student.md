# Lab 07 – VLAN and Trunk Troubleshooting Scenario

## Objective
Practice troubleshooting VLAN and trunk issues using **show** commands and systematic problem‑solving.  
This is a **mystery lab** — the topology is already built, but several configuration mistakes exist.  
Your job is to **find them, fix them, and prove connectivity works**.

---

## Scenario

You are given a Packet Tracer topology:

```
[PC1] -- Fa0/2       Fa0/24       Fa0/24       Fa0/2 -- [PC2]
             SW1 ---------------------- SW2
```

- **PC1** should be in **VLAN 10 (HR)** on SW1  
- **PC2** should be in **VLAN 10 (HR)** on SW2  
- **Fa0/24** between SW1 and SW2 should be a **trunk carrying VLAN 10**

### Current Problem:
**PC1 cannot ping PC2.**

Your job:

1. Document the current state  
2. Identify misconfigurations  
3. Correct them  
4. Verify connectivity  

---

# Step 1. Load the Broken Topology
Open the Packet Tracer file provided by your instructor.

**Do NOT start changing things yet.**  
Your first task is only **information gathering**.

---

# Step 2. Document VLANs and Port Assignments

### On SW1:
```
show vlan brief
show interfaces status
show interfaces fa0/2 switchport
```

### On SW2:
```
show vlan brief
show interfaces status
show interfaces fa0/2 switchport
```

### Questions to answer:
- Does VLAN **10** exist on each switch?
- Which VLAN is **Fa0/2** on SW1 actually in?
- Which VLAN is **Fa0/2** on SW2 in?

Write your answers in your report.

---

# Step 3. Document Trunk Status

### Run on both switches:
```
show interfaces trunk
show interfaces fa0/24 switchport
```

### Questions:
- Is Fa0/24 a trunk on both ends?
- Which VLANs are allowed on the trunk?
- Is VLAN **10** listed under:
  - “Vlans allowed”
  - “Vlans allowed and active in management domain”?
- Is there a **native VLAN mismatch**?

Record all findings.

---

# Step 4. Test Connectivity and Document It

From **PC1**:
```
ping <PC2 IP>
```

Expected right now: **Failure**

Record:
- The exact command  
- The output  
- The time you tested  

This will let you compare before/after results.

---

# Step 5. Fix the Issues (Your Work)

Based on what you found:

### Possible tasks:
- Create missing VLANs
- Correct access port VLANs
- Fix trunk allowed VLANs
- Fix native VLAN if mismatched

### Useful commands:
```
vlan 10
name HR
!
interface fa0/2
 switchport mode access
 switchport access vlan 10
!
interface fa0/24
 switchport mode trunk
 switchport trunk allowed vlan 10
```

**Do NOT copy blindly.**  
Use only the commands that fix the actual problems you observed.

---

# Step 6. Re‑Verify Everything

After you apply corrections, run again:

### On SW1 and SW2:
```
show vlan brief
show interfaces trunk
show interfaces fa0/2 switchport
```

You should now confirm:

✔ VLAN 10 exists on both switches  
✔ PC ports (Fa0/2) are access ports in VLAN 10  
✔ Trunk Fa0/24 allows VLAN 10  
✔ No native VLAN mismatch (unless instructed otherwise)

---

# Step 7. Test Ping Between PCs Again

From PC1:
```
ping <PC2 IP>
```

Expected now: **Successful replies**

If still failing:
- Check PC IP addresses
- Ensure both PCs are in the same subnet
- Check cable connections and link lights

---

# Troubleshooting Report (Deliverable)

Write a short report including:

### 1. **Symptoms observed**
- What was broken?

### 2. **Commands used**
- List all show commands

### 3. **Root causes**
- VLAN missing?  
- Wrong access VLAN?  
- Trunk missing VLAN 10?

### 4. **Fixes applied**
- Include relevant commands

### 5. **Verification**
- Pings  
- Show command output summaries  

---

# Reflection Questions

1. Why must you check VLANs on *both* ends of a trunk?
2. How can a missing VLAN on just one switch break connectivity?
3. Why is `show interfaces trunk` so important?
4. What are your personal **top 3 commands** for VLAN/trunk troubleshooting?

---

# Summary Table

| Concept | Description |
|--------|-------------|
| VLAN mismatch | VLAN missing or wrong on one switch |
| Trunk allowed list | Trunk must allow all user VLANs |
| Access VLAN | End devices must be in correct VLAN |
| Troubleshooting flow | `show vlan brief` → `show interfaces trunk` → `show interfaces switchport` |

---

Good luck
