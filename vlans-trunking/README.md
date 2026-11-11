# Lab 01: VLANs, Trunking and Inter-VLAN Routing
*Simulated scenario in Cisco Packet Tracer*

## Scenario
A small Nairobi office has 6 staff in 3 departments (Accounts, HR, Sales),
2 PCs each. All PCs share one switch and one subnet, so every department can
reach every other and broadcasts hit every device. The owner wants the
departments separated, but still able to reach each other through the router.

## Design
| Department | VLAN | Switch ports | Subnet | PCs              | Gateway |
| Accounts | 10 | Fa0/1-2 | 192.168.10.0/24 | PC1 .11, PC2 .12 | 192.168.10.1 |
| HR       | 20 | Fa0/3-4 | 192.168.20.0/24 | PC3 .11, PC4 .12 | 192.168.20.1 |
| Sales    | 30 | Fa0/5-6 | 192.168.30.0/24 | PC5 .11, PC6 .12 | 192.168.30.1 |

Router-on-a-stick: one trunk (SW1 Gi0/1 to R1 Gi0/0), one subinterface per VLAN.
Native VLAN moved off VLAN 1 to VLAN 99 on both ends of the trunk.



topology:<img width="1894" height="746" alt="topology" src="https://github.com/user-attachments/assets/6fe2cecb-3046-417f-98d4-254893624b7a" />



## Evidence
1. Flat network: all PCs can ping each other (the problem): 1-flat-ping.png '<img width="688" height="577" alt="1-flat ping" src="https://github.com/user-attachments/assets/abeb6d48-ec0f-4c8d-9079-ef684a3364d6" />
`
2. VLANs created: cross-department ping fails (isolation works): ping to diff vlan.png: '<img width="631" height="461" alt="ping to diff vlan" src="https://github.com/user-attachments/assets/e5258970-eea3-4c3a-8567-8efe4f0008da" />
`
3. Router subinterfaces added: cross-VLAN ping works: ping 10 to 20.png,:'<img width="849" height="715" alt="ping 10 to 20" src="https://github.com/user-attachments/assets/88181a51-c52a-4f51-bb51-02e8ee4b2f09" />'
ping vl20 to vl30.png:'<img width="722" height="729" alt="ping vl20 to vl 30" src="https://github.com/user-attachments/assets/e4af1ace-40ff-4c01-976d-b43c2cbf9ce8" />'
ping vl30 to vl10.png :<img width="851" height="727" alt="ping vl 30 - vl 10" src="https://github.com/user-attachments/assets/1b595847-391f-4e49-8b8e-001047bef6ec" />
4. `tracert` PC1 to PC3 shows the router as first hop: `<img width="668" height="178" alt="PC1 tracert" src="https://github.com/user-attachments/assets/b59f6849-6d46-4751-b160-a2e3775f615a" />
PC1 tracert.png`

Full commands and `show` outputs are in notes.txt.

## Key configuration
**SW1**
vlan 10 / name ACCOUNTS, vlan 20 / name HR, vlan 30 / name SALES,
access ports per the design table, `switchport mode trunk` and
`switchport trunk native vlan 99` on Gi0/1.

**R1**
Gi0/0.10, .20 and .30 with `encapsulation dot1Q <vlan>` and the gateway IPs,
plus Gi0/0.99 with `encapsulation dot1Q 99 native`.

## What went wrong
I configured a native VLAN on the subinterface meant for VLAN 10
(`en dot1q 99 native`). Inter-VLAN routing for VLAN 10 stopped working.
I found it by reading the running config, removed the command with `no`, and
restored `encapsulation dot1Q 10` with its IP address.

## What I learned
Never set the native VLAN on a subinterface meant for another VLAN. Give the
native VLAN its own subinterface, and make sure it matches the native VLAN
configured on the switch trunk.
