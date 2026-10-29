# Step-by-step (AWS Console, region ap-southeast-2 Sydney in my run)

Tag every resource: `Project=EWB-VPC-Peering`, `Student=<your name>`. Record every ID in the README inventory.

## 1. VPC A
1. VPC > Your VPCs > Create VPC > **VPC only**: `ewb-vpc-a`, 10.10.0.0/16, no IPv6, tenancy Default.
2. Subnets (same AZ): `ewb-a-public` 10.10.1.0/24, `ewb-a-private` 10.10.2.0/24.
3. Internet gateway `ewb-igw-a` > Actions > Attach to VPC > ewb-vpc-a.
4. Route table `ewb-rt-a-public` (VPC A): associate ewb-a-public; add 0.0.0.0/0 -> ewb-igw-a.
5. Route table `ewb-rt-a-private` (VPC A): associate ewb-a-private; local route only.
**Checkpoint 01:** screenshot associations + public default route.

## 2. VPC B
`ewb-vpc-b` 10.20.0.0/16; subnet `ewb-b-private` 10.20.1.0/24; route table `ewb-rt-b-private` associated explicitly; no IGW; auto-assign public IP off.

## 3. Security groups (inbound)
See `docs/architecture.md`. Keep default outbound allow-all. Never open SSH/ICMP to 0.0.0.0/0. Find your IP: `curl https://checkip.amazonaws.com`.

## 4. EC2 (Amazon Linux 2023, t3.micro)
1. Create RSA key pair `ewb-lab` (.pem) - keep on laptop only.
2. Launch: bastion (ewb-a-public, 10.10.1.10, public IP **enabled**, ewb-sg-bastion); app-a (ewb-a-private, 10.10.2.10, no public IP, ewb-sg-app-a); app-b (ewb-b-private, 10.20.1.10, no public IP, ewb-sg-app-b).
3. Small root disk, delete-on-termination ON.
**Checkpoint 02:** IDs, private IPs, bastion public IP; App-A/B public IPv4 empty.

## 5. SSH access
```bash
mkdir -p ~/.ssh && mv ewb-lab.pem ~/.ssh/ && chmod 400 ~/.ssh/ewb-lab.pem
# add contents of ssh/config.example to ~/.ssh/config (set BASTION_PUBLIC_IP)
ssh ewb-bastion
exit
ssh ewb-app-a
ping -c 4 10.20.1.10     # BASELINE: expect no replies - screenshot it
```

## 6. Peering
Peering connections > Create: name `ewb-a-b-peer`, requester ewb-vpc-a, My account, This Region, accepter ewb-vpc-b. Then **Actions > Accept request**. Wait for Active.
**Checkpoint 03:** screenshot name, Active, both VPC IDs.

## 7. Routes (both sides!)
Add the pcx routes listed in `docs/architecture.md` to all three route tables.
**Checkpoint 04:** screenshot all three tables + associations.

## 8. Verify (record real output)
```bash
# App-A
hostname; hostname -I; ping -c 4 10.20.1.10     # forward
# App-B (from laptop: ssh ewb-app-b)
hostname; hostname -I; ping -c 4 10.10.2.10     # reverse
```
Controlled failure: delete ONLY the ICMP rule on ewb-sg-app-b (source 10.10.2.10/32) -> ping from App-A fails -> re-add exact rule -> ping recovers. Keep SSH rule.
Isolation: EC2 details of App-A and App-B show no public IPv4.

## 9. Cleanup (save evidence first)
1. Terminate 3 instances; confirm volumes gone; release any Elastic IP; delete key pair + local key if unneeded.
2. Delete peer routes, `ewb-a-b-peer`, security groups (after ENIs vanish), disassociate + delete route tables, delete subnets, detach + delete IGW, delete both VPCs.
3. Filter by tag `Project=EWB-VPC-Peering` to confirm nothing remains; check Billing.
