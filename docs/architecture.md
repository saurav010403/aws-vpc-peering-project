# Architecture and design decisions

- **Non-overlapping CIDRs** (10.10.0.0/16 vs 10.20.0.0/16): peering is impossible if VPC CIDRs overlap; different subnet ranges don't fix that.
- **One IGW, only on VPC A**: only the bastion is internet-facing. VPC B has no IGW.
- **Private app hosts**: no public IPs; reachable only via bastion (ProxyJump) or over peering.
- **Peering is not enough**: it needs routes on both sides + security group rules.
- **Non-transitive**: peering gives VPC B no internet via VPC A's IGW.
- **Security groups are stateful**: replies to allowed traffic are automatic. Default NACLs (allow-all) are used.

## Traffic paths
- Admin: laptop -> IGW -> bastion (10.10.1.10) -> App-A (local route) or App-B (peering route).
- Project traffic: App-A <-> App-B via private IPs over `ewb-a-b-peer`.

## Route tables
| Table | Destination | Target |
|---|---|---|
| ewb-rt-a-public | 10.10.0.0/16 | local |
| | 0.0.0.0/0 | igw |
| | 10.20.0.0/16 | pcx |
| ewb-rt-a-private | 10.10.0.0/16 | local |
| | 10.20.0.0/16 | pcx |
| ewb-rt-b-private | 10.20.0.0/16 | local |
| | 10.10.0.0/16 | pcx |

## Security group inbound rules
| SG | Type | Source |
|---|---|---|
| ewb-sg-bastion | SSH 22 | my public IPv4 /32 |
| ewb-sg-app-a | SSH 22 | 10.10.1.10/32 |
| ewb-sg-app-a | ICMP echo request | 10.20.1.10/32 |
| ewb-sg-app-b | SSH 22 | 10.10.1.10/32 |
| ewb-sg-app-b | ICMP echo request | 10.10.2.10/32 |
