# CCNA Lab Day 3 – Advanced Trunking (Native VLAN and Allowed VLANs)

## Overview

This lab focuses on advanced trunking concepts used in enterprise networks. The objective is to understand how VLAN traffic travels between switches, how Native VLANs work, how to restrict VLAN traffic using allowed VLAN lists, and how to troubleshoot common trunk-related issues.

## Objectives

After completing this lab, you will be able to:

- Configure trunk links between switches
- Understand VLAN tagging
- Configure and verify Native VLANs
- Configure allowed VLANs on a trunk
- Troubleshoot Native VLAN mismatches
- Troubleshoot VLAN communication issues

## Topology

```text
                 VLAN 10

PC1 -------- SW1 ===== SW2 -------- PC2


                 VLAN 20

PC3 -------- SW1 ===== SW2 -------- PC4


                 VLAN 30

PC5 -------- SW1 ===== SW2 -------- PC6
```

Trunk Link:

```text
SW1 Fa0/1 ===== SW2 Fa0/1
```

## Devices Used

- 2 Cisco Switches
- 6 PCs
- Copper Straight-Through Cables

## Connections

| Device | Interface | Connected To |
|----------|----------|----------|
| SW1 Fa0/1 | | SW2 Fa0/1 |
| PC1 | Fa0 | SW1 Fa0/2 |
| PC2 | Fa0 | SW2 Fa0/2 |
| PC3 | Fa0 | SW1 Fa0/3 |
| PC4 | Fa0 | SW2 Fa0/3 |
| PC5 | Fa0 | SW1 Fa0/4 |
| PC6 | Fa0 | SW2 Fa0/4 |

## IP Addressing

### VLAN 10 (HR)

| Device | IP Address |
|----------|----------|
| PC1 | 192.168.10.10/24 |
| PC2 | 192.168.10.20/24 |

### VLAN 20 (Finance)

| Device | IP Address |
|----------|----------|
| PC3 | 192.168.20.10/24 |
| PC4 | 192.168.20.20/24 |

### VLAN 30 (Sales)

| Device | IP Address |
|----------|----------|
| PC5 | 192.168.30.10/24 |
| PC6 | 192.168.30.20/24 |

### VLAN 40 (IT)

| Device | IP Address |
|----------|----------|
| PC7 | 192.168.40.10/24 |
| PC8 | 192.168.40.20/24 |

## Configuration Steps

### Step 1: Create VLANs

Configure on both switches:

```bash
enable
configure terminal

vlan 10
name HR

vlan 20
name FINANCE

vlan 30
name SALES

vlan 40
name IT

vlan 999
name NATIVE
```

### Step 2: Configure Access Ports

Example for SW1:

```bash
interface fa0/2
switchport mode access
switchport access vlan 10

interface fa0/3
switchport mode access
switchport access vlan 20

interface fa0/4
switchport mode access
switchport access vlan 30
```

Configure matching VLAN assignments on SW2.

### Step 3: Configure Trunk Ports

On both switches:

```bash
interface fa0/1
switchport mode trunk
```

Verify:

```bash
show interfaces trunk
```

### Step 4: Configure Native VLAN

On both switches:

```bash
interface fa0/1
switchport trunk native vlan 999
```

Verify:

```bash
show interfaces trunk
```

Expected Output:

```text
Native VLAN: 999
```

### Step 5: Configure Allowed VLANs

On both switches:

```bash
interface fa0/1
switchport trunk allowed vlan 10,20,30,40,999
```

Verify:

```bash
show interfaces trunk
```

## Verification Commands

### Verify VLAN Database

```bash
show vlan brief
```

### Verify Trunk Status

```bash
show interfaces trunk
```

### Verify Switchport Details

```bash
show interfaces fa0/1 switchport
```

### Verify Running Configuration

```bash
show running-config
```

## Understanding VLAN Tagging

When traffic crosses a trunk link, the switch adds a VLAN tag to identify which VLAN the frame belongs to.

Example:

```text
VLAN 10 Frame
     |
     v
+----------------+
| VLAN Tag = 10  |
+----------------+
```

The receiving switch reads the VLAN tag and forwards the frame to the correct VLAN.

## Understanding Native VLAN

By default:

```text
Native VLAN = VLAN 1
```

In this lab:

```text
Native VLAN = VLAN 999
```

Frames belonging to the Native VLAN are sent untagged across the trunk.

Example:

```text
VLAN 10 = Tagged
VLAN 20 = Tagged
VLAN 30 = Tagged
VLAN 40 = Tagged
VLAN 999 = Untagged
```

## Connectivity Testing

### Test VLAN 10

```bash
PC1 -> PC2
```

Expected Result:

Success

### Test VLAN 20

```bash
PC3 -> PC4
```

Expected Result:

Success

### Test VLAN 30

```bash
PC5 -> PC6
```

Expected Result:

Success

### Test VLAN 40

```bash
PC7 -> PC8
```

Expected Result:

Success

## Troubleshooting Scenarios

### Scenario 1: Native VLAN Mismatch

SW1:

```bash
interface fa0/1
switchport trunk native vlan 999
```

SW2:

```bash
interface fa0/1
switchport trunk native vlan 100
```

Verify:

```bash
show interfaces trunk
```

Expected Result:

Native VLAN mismatch warning.

### Scenario 2: Remove VLAN 40 from Allowed VLANs

```bash
interface fa0/1
switchport trunk allowed vlan 10,20,30,999
```

Test:

```text
PC7 -> PC8
```

Expected Result:

Communication fails.

Reason:

VLAN 40 is not allowed across the trunk.

### Scenario 3: Restore VLAN 40

```bash
interface fa0/1
switchport trunk allowed vlan 10,20,30,40,999
```

Expected Result:

Communication restored.

### Scenario 4: Remove Trunk Mode

```bash
interface fa0/1
switchport mode access
```

Expected Result:

All inter-switch VLAN communication fails.

## Lab Results

Verification command output:

```text
Switch#show interfaces trunk

Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      999

Port        Vlans allowed on trunk
Fa0/1       10,20,30,40,999

Port        Vlans allowed and active in management domain
Fa0/1       10,20,30,40,999

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,20,30,40,999
```

This confirms:

- Trunk is operational
- Native VLAN is 999
- VLANs 10, 20, 30, 40, and 999 are allowed
- All VLANs are active
- STP is forwarding traffic correctly

## Skills Learned

- VLAN Tagging
- Native VLAN Configuration
- Allowed VLAN Configuration
- Trunk Verification
- Native VLAN Mismatch Troubleshooting
- Allowed VLAN Troubleshooting
- Enterprise Switching Best Practices

## Conclusion

This lab introduced advanced trunking concepts and demonstrated how VLAN traffic is carried between switches using 802.1Q trunking. Understanding Native VLANs, VLAN tagging, and allowed VLANs is essential for building secure and scalable enterprise networks.

## Next Lab

Day 4 – Router-on-a-Stick (Inter-VLAN Routing)

Topics:

- Router Subinterfaces
- 802.1Q Encapsulation
- Default Gateway Configuration
- Inter-VLAN Communication
- Router-on-a-Stick Troubleshooting

## Author

Muhammad Kausar

CCNA SRWE Hands-On Lab Series
