# **Lab 05 -- Inter-VLAN Routing with a Multilayer Switch (SVIs)**

**Objective:**\
Configure inter-VLAN routing using a multilayer switch (MLS) and Switch
Virtual Interfaces (SVIs) instead of router-on-a-stick.

## **Topology**

            [MLS1]  (Multilayer Switch)
            /     \
         Fa0/1   Fa0/2
         /         \
      [SW1]       [SW2]
       |            |
     [PC1]        [PC2]

**Devices:**

-   **MLS1:** Cisco Catalyst 3560 (Multilayer Switch)\
-   **SW1, SW2:** Cisco 2960 access switches\
-   **PC1:** VLAN 10 (HR)\
-   **PC2:** VLAN 20 (FINANCE)

## **VLANs**

-   VLAN 10 -- HR\
-   VLAN 20 -- FINANCE

## **IP Plan**

  Device   VLAN   Interface       IP Address
  -------- ------ --------------- ------------------------------------
  MLS1     10     VLAN 10 (SVI)   192.168.10.1/24
  MLS1     20     VLAN 20 (SVI)   192.168.20.1/24
  PC1      10     NIC             192.168.10.10/24 (GW 192.168.10.1)
  PC2      20     NIC             192.168.20.10/24 (GW 192.168.20.1)

# **Step 1. Build the Topology**

Add 1 multilayer switch (**MLS1**)\
Add 2 access switches (**SW1**, **SW2**)\
Add 2 PCs (**PC1**, **PC2**)

**Cabling:**

-   PC1 → SW1 Fa0/2\
-   PC2 → SW2 Fa0/2\
-   SW1 Fa0/24 → MLS1 Fa0/1\
-   SW2 Fa0/24 → MLS1 Fa0/2

# **Step 2. Create VLANs on All Switches**

### On MLS1

    enable
    configure terminal
    vlan 10
     name HR
    vlan 20
     name FINANCE

### On SW1

    vlan 10
     name HR
    vlan 20
     name FINANCE

### On SW2

    vlan 10
     name HR
    vlan 20
     name FINANCE

# **Step 3. Configure Access Ports for PCs**

### On SW1 (PC1 in VLAN 10):

    interface fa0/2
     switchport mode access
     switchport access vlan 10

### On SW2 (PC2 in VLAN 20):

    interface fa0/2
     switchport mode access
     switchport access vlan 20

# **Step 4. Configure Trunks to the Multilayer Switch**

### On SW1

    interface fa0/24
     switchport mode trunk
     switchport trunk allowed vlan 10,20

### On SW2

    interface fa0/24
     switchport mode trunk
     switchport trunk allowed vlan 10,20

### On MLS1 (3560 requires trunk encapsulation first)

#### Fa0/1

    interface fa0/1
     switchport trunk encapsulation dot1q
     switchport mode trunk
     switchport trunk allowed vlan 10,20

#### Fa0/2

    interface fa0/2
     switchport trunk encapsulation dot1q
     switchport mode trunk
     switchport trunk allowed vlan 10,20

Verify:

    show interfaces trunk

# **Step 5. Configure SVIs and Enable Routing on MLS1**

### Create SVIs:

    interface vlan 10
     ip address 192.168.10.1 255.255.255.0
     no shutdown

    interface vlan 20
     ip address 192.168.20.1 255.255.255.0
     no shutdown

### Enable Layer 3 Routing:

    ip routing

Verify:

    show ip interface brief
    show ip route

# **Expected:**

-   VLAN 10 and VLAN 20 SVIs up/up\
-   Connected routes for 192.168.10.0/24 and 192.168.20.0/24

# **Step 6. Configure PCs**

### PC1 -- VLAN 10

-   IP: 192.168.10.10\
-   Mask: 255.255.255.0\
-   Gateway: 192.168.10.1

### PC2 -- VLAN 20

-   IP: 192.168.20.10\
-   Mask: 255.255.255.0\
-   Gateway: 192.168.20.1

# **Step 7. Test Inter-VLAN Connectivity**

From PC1:

    ping 192.168.20.10

From PC2:

    ping 192.168.10.10

If pings fail, check:

-   show vlan brief\
-   show interfaces trunk\
-   show ip interface brief\
-   show ip route

# **Verification Checklist**

  ----------------------------------------------------------------------------
  Device       Check          Command                      Expected
  ------------ -------------- ---------------------------- -------------------
  MLS1         SVIs Up        show ip interface brief      VLAN 10/20 up/up

  MLS1         Routing        show ip route                Connected routes
                                                           exist

  SW1, SW2,    Trunks         show interfaces trunk        VLANs 10,20 allowed
  MLS1                                                     

  SW1, SW2     PC Ports       show interfaces fa0/2        Correct VLAN access
                              switchport                   

  PCs          Connectivity   ping                         Success
  ----------------------------------------------------------------------------

# **Reflection Questions**

-   How is this design different from router-on-a-stick?\
-   What role do SVIs play in this network?\
-   Why must we enable `ip routing` on the multilayer switch?\
-   What happens if VLAN 10 is missing on SW1? (Hint: PC1's frames will
    never reach MLS1.)

# **Summary**

  -----------------------------------------------------------------------
  Concept            Description
  ------------------ ----------------------------------------------------
  **SVI**            Layer 3 interface bound to a VLAN

  **Multilayer       A switch that can also route
  Switch**           

  **Inter-VLAN       Done directly on MLS via SVIs
  Routing**          

  **Verification**   show ip int brief, show ip route, show interfaces
                     trunk
  -----------------------------------------------------------------------
