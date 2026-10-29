# AWS VPC Peering - Project Report
**Name:** Saurav Mani Tripathi | **Batch:** ____ | **Region:** ap-southeast-2 | **Completion date:** ____ | **GitHub:** https://github.com/saurav010403/aws-vpc-peering-project

## 1. Objective
Connect two private VPCs over a peering connection, administer via a bastion, and prove connectivity with positive and negative tests.

## 2. Architecture and inventory
(diagram + table with actual IDs and private IPs)

## 3. Subnets, IGW and route tables
(screenshots: subnet CIDRs, all three route tables with associations)

## 4. Security groups
(inbound rules of each SG, with reason for each)

## 5. Peering
(screenshot: ewb-a-b-peer Active, both VPC IDs)

## 6. Tests
| Test | Host | Command | Result | Screenshot |
|---|---|---|---|---|
| Baseline (before peering) | App-A | ping -c 4 10.20.1.10 | | |
| Forward | App-A | ping -c 4 10.20.1.10 | | |
| Reverse | App-B | ping -c 4 10.10.2.10 | | |
| Private SSH | laptop | ssh ewb-app-b | | |
| Isolation | console | no public IPv4 | | |
| Controlled failure + recovery | App-A | ping after rule removal / restore | | |

## 7. Forward path and return path
App-A -> ewb-rt-a-private (10.20.0.0/16 -> pcx) -> peering -> App-B (SG allows ICMP from 10.10.2.10/32). Reply -> ewb-rt-b-private (10.10.0.0/16 -> pcx) -> App-A (stateful SG allows the reply).

## 8. Troubleshooting faced
(real issues and fixes)

## 9. Cleanup evidence
(screenshots)

## 10. What I learned
(your own words)
