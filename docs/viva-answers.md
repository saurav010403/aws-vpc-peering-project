# Viva: be ready to explain
1. **Why a return route?** Peering is a two-way link but routing is per subnet. Without 10.10.0.0/16 -> pcx in VPC B's table, App-B has no path to send replies, so the connection fails even though requests arrive.
2. **Why is the App-B SSH source the bastion private IP?** ProxyJump makes the bastion open a new TCP connection to App-B; App-B sees the bastion's private address 10.10.1.10, not my laptop.
3. **Why doesn't Active peering guarantee ping?** Active only means the link exists. Route tables must point to it on both sides and security groups must allow ICMP.
4. **Can VPC B use VPC A's IGW over peering?** No. Peering is non-transitive; edge routing through the peer is not supported.
5. **Why does the public subnet need a pcx route?** The bastion initiates the SSH to App-B; the more specific 10.20.0.0/16 route beats 0.0.0.0/0.
6. **Why no NAT gateway?** Private hosts need no internet; avoids cost.
7. **Why /32 sources?** Least privilege: only specific lab hosts are allowed.
