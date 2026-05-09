# VLSM with OSPF – Network Diagram Implementation

This repository shows a **VLSM** design for five networks (100, 50, 25, 10, 5 hosts) inside `192.168.10.0/24`, and then configures **OSPF** routing according to the physical topology shown in the diagram (`312429.jpg`).

## 📡 Topology (as per diagram)

The diagram consists of **three routers** interconnected via serial links, with end‑user subnets attached to FastEthernet ports.

### Router1 (R1)
- `Fa0/0` → **Net F** – 5 hosts → `192.168.10.240/29`
- `Se0/0/0` → link to **Router2** `Se0/0/0`

### Router2 (R2)
- `Fa0/0` → **Net D** – 10 hosts → `192.168.10.224/28`
- `Fa0/1` → **Net B** – 50 hosts → `192.168.10.128/26`
- `Fa0/2` → **Net C** – 25 hosts → `192.168.10.192/27`
- `Fa0/3` → (spare / future use)
- `Fa0/4` → (spare)
- `Se0/0/0` → link to **Router1** `Se0/0/0`
- `Se0/1/1` → link to **Router3** `Se0/1/1`

### Router3 (R3)
- `Fa0/0` → **Net A** – 100 hosts → `192.168.10.0/25`
- `Fa0/1` → (spare)
- `Se0/1/1` → link to **Router2** `Se0/1/1`

> *Note*: The diagram also shows `Fa0/4` and other ports on R2, which are reserved for future expansion (not used in OSPF config).

## 🧮 VLSM Subnet Summary (as calculated)

| Network | Required hosts | CIDR | Subnet mask         | Network address | Usable range                         | Broadcast     |
|---------|----------------|------|---------------------|-----------------|--------------------------------------|---------------|
| A       | 100            | /25  | 255.255.255.128     | 192.168.10.0    | 192.168.10.1   – 192.168.10.126      | 192.168.10.127|
| B       | 50             | /26  | 255.255.255.192     | 192.168.10.128  | 192.168.10.129 – 192.168.10.190      | 192.168.10.191|
| C       | 25             | /27  | 255.255.255.224     | 192.168.10.192  | 192.168.10.193 – 192.168.10.222      | 192.168.10.223|
| D       | 10             | /28  | 255.255.255.240     | 192.168.10.224  | 192.168.10.225 – 192.168.10.238      | 192.168.10.239|
| F       | 5              | /29  | 255.255.255.248     | 192.168.10.240  | 192.168.10.241 – 192.168.10.246      | 192.168.10.247|

## 🔌 Interface IP Addressing

We assign the **first usable IP** of each subnet to the router interface.

| Router | Interface | Connected to | IP address            | Subnet mask         |
|--------|-----------|--------------|-----------------------|---------------------|
| R1     | Fa0/0     | Net F        | 192.168.10.241        | 255.255.255.248     |
| R1     | Se0/0/0   | R2 Se0/0/0   | 192.168.10.249 /30    | 255.255.255.252     |
| R2     | Fa0/0     | Net D        | 192.168.10.225        | 255.255.255.240     |
| R2     | Fa0/1     | Net B        | 192.168.10.129        | 255.255.255.192     |
| R2     | Fa0/2     | Net C        | 192.168.10.193        | 255.255.255.224     |
| R2     | Se0/0/0   | R1 Se0/0/0   | 192.168.10.250 /30    | 255.255.255.252     |
| R2     | Se0/1/1   | R3 Se0/1/1   | 192.168.10.253 /30    | 255.255.255.252     |
| R3     | Fa0/0     | Net A        | 192.168.10.1          | 255.255.255.128     |
| R3     | Se0/1/1   | R2 Se0/1/1   | 192.168.10.254 /30    | 255.255.255.252     |

> *Point‑to‑point links* use a `/30` mask (4 addresses, 2 usable). We used the first available `/30` block after the VLSM subnets: `192.168.10.248/30` (R1–R2) and `192.168.10.252/30` (R2–R3). These do not overlap with the five subnets.

## 🧭 OSPF Configuration (Cisco style)

All interfaces are placed in **OSPF area 0**. Below are the exact commands per router.

### Router1 (R1)
```

interface FastEthernet0/0
ip address 192.168.10.241 255.255.255.248
ip ospf 1 area 0
!
interface Serial0/0/0
ip address 192.168.10.249 255.255.255.252
ip ospf 1 area 0
clock rate 64000   ! DCE side (if needed)
!
router ospf 1
router-id 1.1.1.1

```

### Router2 (R2)
```

interface FastEthernet0/0
ip address 192.168.10.225 255.255.255.240
ip ospf 1 area 0
!
interface FastEthernet0/1
ip address 192.168.10.129 255.255.255.192
ip ospf 1 area 0
!
interface FastEthernet0/2
ip address 192.168.10.193 255.255.255.224
ip ospf 1 area 0
!
interface Serial0/0/0
ip address 192.168.10.250 255.255.255.252
ip ospf 1 area 0
!
interface Serial0/1/1
ip address 192.168.10.253 255.255.255.252
ip ospf 1 area 0
clock rate 64000   ! DCE side (if needed)
!
router ospf 1
router-id 2.2.2.2

```

### Router3 (R3)
```

interface FastEthernet0/0
ip address 192.168.10.1 255.255.255.128
ip ospf 1 area 0
!
interface Serial0/1/1
ip address 192.168.10.254 255.255.255.252
ip ospf 1 area 0
!
router ospf 1
router-id 3.3.3.3

```

### Alternative: Network‑statement method

If you prefer the classic `network` commands with wildcard masks:

**On all routers** (same OSPF process 1):
```

router ospf 1
network 192.168.10.0 0.0.0.127 area 0      ! Net A
network 192.168.10.128 0.0.0.63 area 0     ! Net B
network 192.168.10.192 0.0.0.31 area 0     ! Net C
network 192.168.10.224 0.0.0.15 area 0     ! Net D
network 192.168.10.240 0.0.0.7 area 0      ! Net F
network 192.168.10.248 0.0.0.3 area 0      ! R1–R2 link
network 192.168.10.252 0.0.0.3 area 0      ! R2–R3 link

```

## Author ✍️
Eng ~ Hani Ahmed Abdullah Muhammad.
