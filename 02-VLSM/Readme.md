# Lab 02: VLSM Re-addressing
*Simulated scenario in Cisco Packet Tracer*

## Scenario (Ticket #002)
The Nairobi office outgrew its flat 192.168.x.x plan, which wastes addresses.
The owner wants addresses sized to each department using one block,
192.168.50.0/24, with room left for a future 30-host VLAN.

![topology](topology.png)

## Addressing plan
| Segment (hosts) | VLAN | Network | Usable | Broadcast | Gateway |
| Sales (50)      | 10 | 192.168.50.0/26 | .1-.62 | .63 | .1 |
| Accounts (25)   | 20 | 192.168.50.64/27 | .65-.94 | .95 | .65 |
| HR (12)         | 30 | 192.168.50.96/28 | .97-.110 | .111 | .97 |
| Servers (5)     | 40 | 192.168.50.112/29 | .113-.118 | .119 | .113 |
| Link to R2      | n/a | 192.168.50.120/30 | .121-.122 | .123 | R1 .121, R2 .122 |

**Future expansion:** 192.168.50.128/26, usable .129-.190, gateway .129.
A /27 gives only 30 usable addresses, which is one short once the gateway
takes one, so a /26 is the smallest that fits.
**Gap:** 192.168.50.124/30 (.125-.126) is unused.
**Leftover:** 192.168.50.192/26 (.193-.254).
**Tradeoff:** the Servers /29 has exactly 5 usable hosts and no growth room.

## Design notes
- Router-on-a-stick: one subinterface per VLAN on R1, plus a native VLAN
  subinterface with no IP address, because no hosts live in it.
- A second switch (SW2) carries VLAN 40 and the servers. SW1 and SW2 are
  joined by a trunk, and the native VLAN matches on both ends.
- R1 has a default route to R2. R2 has one static summary route for
  192.168.50.0/24 pointing back to R1.
- R2 simulates an upstream router. A real ISP link would need NAT for private
  addresses, which I'll cover in a later lab.

## Configuration evidence
SW1 and SW2:
![SW2 VLANs]:(<img width="784" height="277" alt="sw2 vlans" src="https://github.com/user-attachments/assets/20ecc400-e3ac-4809-8d20-1b578efd6b6a" />
sw2 vlans.png)
![SW2 trunk]:(<img width="725" height="239" alt="SW2 g01 int tr " src="https://github.com/user-attachments/assets/375e3f75-8c2e-4b77-9809-d1fe1b7e757e" />
sw2 g01 int tr.png)
![SW1 native VLAN](<img width="669" height="91" alt="sw1 native vlan" src="https://github.com/user-attachments/assets/0f65d9e1-738a-4f4b-8e93-1215c298c38a" />
sw1-native-vlan.png)
![SW2 native VLAN](<img width="678" height="68" alt="sw2 native vl" src="https://github.com/user-attachments/assets/03c4f6f6-5ebb-4b59-b228-647685b65262" />
sw2-native-vlan.png)

R1:
![R1 subinterfaces](<img width="479" height="310" alt="R1 subinterfaces" src="https://github.com/user-attachments/assets/322b6c63-d952-42dd-af66-bb9e759f5a8f" />
r1-subinterfaces.png)
![R1 native subinterface](<img width="416" height="49" alt="R1 native vl" src="https://github.com/user-attachments/assets/f9a00bbe-d318-4f32-847c-917fc1576a84" />
r1-native-subinterface.png)
![R1 default route](<img width="436" height="28" alt="default ip route" src="https://github.com/user-attachments/assets/c840f619-c95a-4c5a-8889-48afc1127159" />
default ip route.png)
![R1 routing table](<img width="775" height="307" alt="R1 routing table" src="https://github.com/user-attachments/assets/232ab6a6-ad95-49bb-a7b3-7999e7f7254e" />
r1-routing-table.png)

R2:
![R2 summary route](<img width="760" height="121" alt="R2 route summ" src="https://github.com/user-attachments/assets/52ffcaf4-3f70-4102-b775-9ff04586109e" />
R2 route summ.png)
![R2 routing table](<img width="718" height="157" alt="R2 rotuing table" src="https://github.com/user-attachments/assets/cad1da62-2dbf-4aa2-b266-128a3b034bd2" />
r2 routing table.png)

## Connectivity tests
| # | From | To | Result |
|---|---|---|---|
| 1 | Sales | Accounts | Pass |
| 2 | Sales | HR | Pass |
| 3 | Sales | Servers | Pass |
| 4 | Sales | R2 | Pass |
| 5 | Accounts | HR | Pass |
| 6 | Accounts | Servers | Pass |
| 7 | Accounts | R2 | Pass |
| 8 | HR | Servers | Pass |
| 9 | HR | R2 | Pass |
| 10 | Servers | R2 | Pass |

Screenshots: ![1](<img width="631" height="291" alt="vlan 10 to vl 20" src="https://github.com/user-attachments/assets/c48fd288-f6b2-4725-bbc2-541bcddf16f9" />
ping-sales-to-acc.png) ![2](<img width="671" height="253" alt="sales to HR" src="https://github.com/user-attachments/assets/ab875fe8-e073-4418-9b38-c906b365db43" />
ping-sales-to-hr.png)
![3](<img width="610" height="292" alt="ping sales to SERvers" src="https://github.com/user-attachments/assets/6225b95a-ad56-4ce0-8567-6a377dac8b0c" />
ping-sales-to-servers.png) ![4](<img width="614" height="263" alt="SALES to R2" src="https://github.com/user-attachments/assets/a0092cf5-fecd-4571-b92e-4120b79186e5" />
ping-sales-to-r2.png)
![5](<img width="648" height="259" alt="acc to hr" src="https://github.com/user-attachments/assets/750b9dd1-f22d-427b-91b1-f4d34c23b1c9" />
ping-acc-to-hr.png) ![6](<img width="657" height="268" alt="acc to servers" src="https://github.com/user-attachments/assets/b1b444c9-ab2d-4868-b99e-49a7745368fb" />
ping-acc-to-servers.png)
![7](<img width="651" height="256" alt="acc to R2" src="https://github.com/user-attachments/assets/13e2b82e-c970-496d-a34c-8a6df66e9694" />
ping-acc-to-r2.png) ![8](<img width="625" height="264" alt="hr to servers" src="https://github.com/user-attachments/assets/55be04eb-7f64-4e8f-b7f5-a050abecdadd" />
ping-hr-to-servers.png)
![9](<img width="627" height="261" alt="hr to R2" src="https://github.com/user-attachments/assets/c3c160b3-9cc2-41d2-85b5-49887391de76" />
ping-hr-to-r2.png) ![10](<img width="660" height="264" alt="server to R2" src="https://github.com/user-attachments/assets/5c5881a1-8ef9-4265-95e3-6eea19fa86d0" />
ping-servers-to-r2.png)

## Problems and fixes
**1. Routing loop to an unused address**
- Symptom: a trace to 192.168.50.200 bounced between R1 and R2.
- Cause: R1's default route sent unknown addresses to R2, and R2's summary
  route sent the whole /24 back to R1.
- Fix: a Null0 route for 192.168.50.0/24 on R1. The more specific real subnets
  still win by longest prefix match.

Before:
![loop](<img width="674" height="273" alt="tracert loop with no null0" src="https://github.com/user-attachments/assets/a16f0ed8-dbd0-4bd2-9114-e164f2482fae" />
trace-loop-before.png)
After:
![fixed](<img width="580" height="234" alt="tracert fix" src="https://github.com/user-attachments/assets/ed528b50-982a-4c3a-96a6-0a80bef6974a" />
trace-loop-after.png)

**2. Only VLAN 40 showed on SW2**
- Cause: I forgot to create VLANs 10, 20 and 30 on SW2.
- Found it with `show vlan brief`. Fix: created them on SW2.

**3. Native VLAN addressing**
I wondered whether the native VLAN needed an IP address. It doesn't: it
decides which VLAN's traffic crosses the trunk untagged.

## What I learned
- A subnet must start on a multiple of its block size, so I can't always use
  the next free address.
- A summary route needs a Null0 partner on the other router, or unused
  addresses loop.
- VLANs must exist on every switch the trunk carries them to.

Full notes: [notes.txt](notes(vlsm).txt)
