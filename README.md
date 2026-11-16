*This project has been created as part of the 42 curriculum by archowdh.*

# NetPractice

## Description

NetPractice is a practical introduction to computer networking. It consists of
**10 levels**, each showing a small simulated network that does not work. The goal
is to edit the editable (unshaded) fields — IP addresses, subnet masks and routing
table entries — until every device listed in the level's goals can communicate.

The networks are simulated in a training interface that runs in a web browser; no
real network is touched. Solving the levels requires understanding:

- how IPv4 addresses and subnet masks define a network,
- which addresses are valid for a host in a given subnet,
- how switches and routers connect devices,
- how routing tables and default gateways send packets between networks,
- that communication must work in **both** directions (the reply needs a route back).

This repository contains the 10 exported configurations (`level1.json` to
`level10.json`), the training interface, and my study notes.

## Instructions

### Running the training interface

The interface is in [net_practice.1.9/net_practice/](net_practice.1.9/net_practice/).

```sh
cd net_practice.1.9/net_practice
./run.sh
```

`run.sh` starts a local web server and opens the training page in your browser.
If it does not work, start the server manually and open the page yourself:

```sh
cd net_practice.1.9/net_practice
python3 -m http.server 49242
# then browse to http://localhost:49242
```

A local web server is required because browsers block some features on pages
opened directly from the filesystem.

### Using the interface

1. On the **Training** tab, enter your intranet login (here: `archowdh`). The login
   selects your personal configuration, so it must be entered before exporting.
   The **Evaluation** tab generates a random configuration instead (used during
   the defense).
2. Read the goals at the top of the level and edit the unshaded fields.
3. Click **Check again** to test the configuration. The log panel at the bottom
   explains failures (e.g. *destination does not match any route*).
4. When the level passes, a **Next level** button appears.

### Exporting configurations

Before moving to the next level, click **Get my config**. This downloads a JSON
file with the current configuration of that level. Rename it if needed so each
level has one file.

### Submission requirements

- **10 exported configuration files, one per level, must be placed at the root of
  the repository**: `level1.json`, `level2.json`, …, `level10.json`.
- The files must be exported with your login entered in the interface.
- During the defense, three random levels must be solved within a time limit,
  without external tools (a simple calculator such as `bc` is tolerated).

## Subnetting reference

A full cheat sheet is in [README-subnetting.md](README-subnetting.md) and longer
notes are in [NETPRACTICE.md](NETPRACTICE.md). The essentials:

### CIDR

`/n` is the number of network bits out of 32.

- Host bits = `32 - n`
- Total addresses = `2 ^ host bits`
- Usable hosts = total − 2 (network and broadcast addresses are excluded)
- Exceptions: `/31` (point-to-point links) and `/32` (a single host route)

Example: `/26` → 6 host bits → 64 addresses → 62 usable hosts.

### Mask chart

| CIDR | Mask              | Increment | Addresses |
|------|-------------------|----------:|----------:|
| /8   | `255.0.0.0`       | 1 (1st octet) | 16 777 216 |
| /16  | `255.255.0.0`     | 1 (2nd octet) | 65 536 |
| /17  | `255.255.128.0`   | 128 (3rd) | 32 768 |
| /18  | `255.255.192.0`   | 64 (3rd)  | 16 384 |
| /19  | `255.255.224.0`   | 32 (3rd)  | 8 192 |
| /20  | `255.255.240.0`   | 16 (3rd)  | 4 096 |
| /21  | `255.255.248.0`   | 8 (3rd)   | 2 048 |
| /22  | `255.255.252.0`   | 4 (3rd)   | 1 024 |
| /23  | `255.255.254.0`   | 2 (3rd)   | 512 |
| /24  | `255.255.255.0`   | 1 (3rd)   | 256 |
| /25  | `255.255.255.128` | 128       | 128 |
| /26  | `255.255.255.192` | 64        | 64 |
| /27  | `255.255.255.224` | 32        | 32 |
| /28  | `255.255.255.240` | 16        | 16 |
| /29  | `255.255.255.248` | 8         | 8 |
| /30  | `255.255.255.252` | 4         | 4 |
| /31  | `255.255.255.254` | 2         | 2 |
| /32  | `255.255.255.255` | 1         | 1 |

Bit weights for a partial octet: `128 64 32 16 8 4 2 1`. Add the weights of the
`1` bits — e.g. `/27` → `11100000` → `128 + 64 + 32 = 224`.

### Finding a subnet in four steps

1. **Mask** from the CIDR prefix.
2. **Increment** (block size) = `256 − partial mask value`.
3. **Block**: list multiples of the increment in that octet and pick the one
   containing the IP.
4. **Range**: network = start of the block, broadcast = end of the block, usable
   hosts = everything in between.

Example — `192.168.1.70/26`:

| Step | Result |
|---|---|
| Mask | `255.255.255.192` |
| Increment | `256 − 192 = 64` → blocks `0, 64, 128, 192` |
| Block | `70` is in `64–127` |
| Network | `192.168.1.64` |
| Usable hosts | `192.168.1.65` – `192.168.1.126` |
| Broadcast | `192.168.1.127` |

Example — `172.16.35.100/20`: mask `255.255.240.0`, increment 16 in the **third**
octet, `35` is in `32–47` → network `172.16.32.0`, broadcast `172.16.47.255`,
usable `172.16.32.1` – `172.16.47.254`.

### Rules used to solve the levels

- **Same link (cable or switch)**: interfaces must be in the same subnet, use the
  same mask, have unique IPs, and not use the network or broadcast address.
- **Switches** have no IP; everything plugged into one switch is one subnet.
- **Router interfaces** each sit in a different subnet, and those subnets must not
  overlap (e.g. `10.0.0.0/24` overlaps `10.0.0.128/25`).
- **Routes** have the form `destination/mask => next hop`. The destination must be
  a network address, and the next hop must be directly reachable (in one of the
  device's own subnets).
- **Default route** `0.0.0.0/0` (written `default`) matches everything; the
  **default gateway** is the router interface on the host's own subnet.
- **Longest prefix match**: the most specific matching route wins.
- **Return trip**: every device on the reply path also needs a route back.
- **Internet**: hosts reaching the Internet need public IPs; the private ranges
  `10.0.0.0/8`, `172.16.0.0/12` and `192.168.0.0/16` are not routed there.
  Never use `127.0.0.0/8` (loopback) or addresses from `224.0.0.0` upward.

## Resources

### Networking concepts studied

- **TCP/IP addressing** — IPv4 structure, binary representation, private/public
  and reserved ranges
- **Subnet masks and CIDR** — network/broadcast addresses, usable host ranges,
  splitting a range into non-overlapping subnets
- **Default gateways** and **routing tables** — next hops, default routes,
  longest prefix match, return routes
- **Routers and switches** — layer 3 vs layer 2 devices
- **OSI layers** — physical, data link, network, transport, application, and how
  they map to the TCP/IP model

### References

- [RFC 791 — Internet Protocol](https://www.rfc-editor.org/rfc/rfc791)
- [RFC 1918 — Address Allocation for Private Internets](https://www.rfc-editor.org/rfc/rfc1918)
- [RFC 4632 — Classless Inter-domain Routing (CIDR)](https://www.rfc-editor.org/rfc/rfc4632)
- [Cloudflare — What is a subnet?](https://www.cloudflare.com/learning/network-layer/what-is-a-subnet/)
- [Cloudflare — What is the OSI model?](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
- [Practical Networking — Subnetting mastery](https://www.practicalnetworking.net/series/subnetting/subnetting/)
- [subnet-calculator.com](https://www.subnet-calculator.com/) — for checking answers while practising
- My notes: [NETPRACTICE.md](NETPRACTICE.md) and [README-subnetting.md](README-subnetting.md)

### Use of AI

AI was used as a study and writing aid, not to solve the levels:

- **Explanations**: asking for explanations of subnetting, routing tables and
  OSI layers while studying, then checking them against the references above and
  by testing in the training interface.
- **Notes and documentation**: help structuring and formatting the study notes,
  the subnetting cheat sheet, and this README.

All 10 levels were solved by hand in the training interface, and every
configuration was verified with **Check again** before exporting.
