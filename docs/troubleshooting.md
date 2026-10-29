# Troubleshooting
Check in order: instance state -> destination IP -> subnet route table -> security rule.

| Symptom | Inspect |
|---|---|
| Bastion SSH times out | bastion public IP; your current IP /32 in ewb-sg-bastion; public subnet association; 0.0.0.0/0 -> IGW |
| App-A SSH fails | private IP; TCP 22 from 10.10.1.10/32; key, `ec2-user` |
| App-B SSH fails | public RT has 10.20.0.0/16 -> pcx; VPC B has return route; SG source is bastion private IP |
| Ping fails after acceptance | both private route tables; correct pcx, Active; ICMP echo rule on destination; real host IPs |
| Peering creation fails | overlapping CIDRs incl. secondary |
| Permission denied (publickey) | IdentityFile, key pair, `ec2-user` (routing is fine) |
| Custom NACLs | both directions, ICMP, ephemeral ports (lab assumes default allow-all) |
| Home IP changed | update ewb-sg-bastion source |
