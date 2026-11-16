# IPv4 Subnetting Cheat Sheet

A concise reference for CIDR notation, subnet masks, network increments, and subnet calculations.

## 1. CIDR in brief

CIDR (Classless Inter-Domain Routing) uses `/n` to show how many of the 32 IPv4 bits belong to the network prefix.

- **Network bits** = CIDR prefix
- **Host bits** = `32 - CIDR prefix`
- **Total IPv4 addresses** = `2 ^ host bits`
- For most ordinary IPv4 subnets, **usable hosts** = `total addresses - 2` (excluding the network and broadcast addresses).

Example: `/26` means 26 network bits and `32 - 26 = 6` host bits. Therefore, it contains `2^6 = 64` addresses, usually 62 usable host addresses.

> Special cases: `/31` is commonly used on point-to-point links, and `/32` represents a single address/host route. Do not apply the usual “subtract two” rule blindly to these prefixes.

## 2. Subnet mask chart

The chart below is written in Markdown, so no image file is needed.

### Prefixes `/17` to `/24` — partial mask is in the third octet

| CIDR | Network bits | Host bits | Increment | Subnet mask |
|---|---:|---:|---:|---|
| `/17` | 17 | 15 | 128 | `255.255.128.0` |
| `/18` | 18 | 14 | 64 | `255.255.192.0` |
| `/19` | 19 | 13 | 32 | `255.255.224.0` |
| `/20` | 20 | 12 | 16 | `255.255.240.0` |
| `/21` | 21 | 11 | 8 | `255.255.248.0` |
| `/22` | 22 | 10 | 4 | `255.255.252.0` |
| `/23` | 23 | 9 | 2 | `255.255.254.0` |
| `/24` | 24 | 8 | 1 in the fourth octet | `255.255.255.0` |

### Prefixes `/25` to `/32` — partial mask is in the fourth octet

| CIDR | Network bits | Host bits | Increment | Subnet mask |
|---|---:|---:|---:|---|
| `/25` | 25 | 7 | 128 | `255.255.255.128` |
| `/26` | 26 | 6 | 64 | `255.255.255.192` |
| `/27` | 27 | 5 | 32 | `255.255.255.224` |
| `/28` | 28 | 4 | 16 | `255.255.255.240` |
| `/29` | 29 | 3 | 8 | `255.255.255.248` |
| `/30` | 30 | 2 | 4 | `255.255.255.252` |
| `/31` | 31 | 1 | 2 | `255.255.255.254` |
| `/32` | 32 | 0 | 1 | `255.255.255.255` |

### Bit values used to calculate a partial mask octet

| Bit | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Decimal weight | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

A subnet mask has network bits set to `1`, followed by host bits set to `0`. Add the weights of the `1` bits to get the decimal value of a partial octet.

Examples:
- `/25`: `10000000` = `128` → `255.255.255.128`
- `/26`: `11000000` = `128 + 64 = 192` → `255.255.255.192`
- `/27`: `11100000` = `128 + 64 + 32 = 224` → `255.255.255.224`
- `/28`: `11110000` = `240` → `255.255.255.240`

## 3. Network increment (block size)

The increment tells you the distance between subnet boundaries in the octet where subnetting occurs.

**Formula:**

`Increment = 256 - partial subnet-mask value`

For example, `/26` has the mask `255.255.255.192`, so its increment is `256 - 192 = 64`. Its fourth-octet subnet boundaries are `0, 64, 128, 192`.

For `/20`, the mask is `255.255.240.0`, so the increment is `256 - 240 = 16` in the third octet.

## 4. Calculate a subnet in four steps

Given an IP address with a CIDR prefix:

1. **Find the mask** from the CIDR prefix.
2. **Find the increment**: `256 - partial mask value`.
3. **Find the subnet boundary**: list multiples of the increment in the relevant octet and choose the block containing the IP.
4. **Calculate the range**:
   - Network address = first address in the block
   - Broadcast address = last address in the block
   - Usable host range = network address + 1 through broadcast address - 1 (for ordinary subnets)

### Example A: `192.168.1.70/26`

- Mask: `255.255.255.192`
- Increment: `256 - 192 = 64`
- Fourth-octet boundaries: `0, 64, 128, 192`
- `70` falls in the `64–127` block.

| Result | Address |
|---|---|
| Network | `192.168.1.64` |
| First usable host | `192.168.1.65` |
| Last usable host | `192.168.1.126` |
| Broadcast | `192.168.1.127` |

### Example B: `192.168.1.150/26`

The same boundaries apply (`0, 64, 128, 192`). Since `150` falls in the `128–191` block:

- Network: `192.168.1.128`
- Usable range: `192.168.1.129–192.168.1.190`
- Broadcast: `192.168.1.191`

### Example C: `172.16.35.100/20`

- Mask: `255.255.240.0`
- Increment: `256 - 240 = 16` in the **third** octet
- Third-octet boundaries: `0, 16, 32, 48, 64, ...`
- `35` falls in the `32–47` block.

| Result | Address |
|---|---|
| Network | `172.16.32.0` |
| First usable host | `172.16.32.1` |
| Last usable host | `172.16.47.254` |
| Broadcast | `172.16.47.255` |

## 5. Quick memory rules

- **CIDR prefix increases** → more network bits, fewer host bits, smaller subnets.
- **Mask value** = sum of the bit weights set to `1`.
- **Increment** = `256 - partial mask value`.
- Apply the increment to the octet containing the partial mask value.
- The network address is the start of the block; the broadcast address is the end.
- Check both IP address and subnet mask when deciding whether two devices are in the same subnet.
