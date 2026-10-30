# Connection and validation tests

## 1. Prepare local SSH configuration

Keep your EC2 private key only on your laptop. On Linux/macOS/WSL, set restrictive permissions:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
chmod 400 ~/.ssh/YOUR_KEY_FILE.pem
```

Use the public IP printed by `terraform output -raw bastion_public_ip` and create entries like the following in `~/.ssh/config`. Replace `BASTION_PUBLIC_IP` and `YOUR_KEY_FILE.pem` with your real values.

```sshconfig
Host ewb-bastion
    HostName BASTION_PUBLIC_IP
    User ec2-user
    IdentityFile ~/.ssh/YOUR_KEY_FILE.pem
    IdentitiesOnly yes

Host ewb-app-a
    HostName 10.10.2.10
    User ec2-user
    IdentityFile ~/.ssh/YOUR_KEY_FILE.pem
    IdentitiesOnly yes
    ProxyJump ewb-bastion

Host ewb-app-b
    HostName 10.20.1.10
    User ec2-user
    IdentityFile ~/.ssh/YOUR_KEY_FILE.pem
    IdentitiesOnly yes
    ProxyJump ewb-bastion
```

Test access:

```bash
ssh ewb-bastion
ssh ewb-app-a
ssh ewb-app-b
```

## 2. Connectivity checks

Run these commands from inside the indicated private instance. Record the actual output in `evidence/`.

On App-A:

```bash
hostname
hostname -I
ping -c 4 10.20.1.10
```

On App-B:

```bash
hostname
hostname -I
ping -c 4 10.10.2.10
```

The expected success criterion after routes and security rules are configured is successful two-way private-IP ping. ICMP rules and routes must both permit traffic. Security groups are stateful; the workbook assumes default network ACLs.

## 3. Controlled failure and recovery

In the AWS console, remove only the inbound ICMP rule on `ewb-sg-app-b` whose source is `10.10.2.10/32`. Keep SSH ingress in place. Re-run the App-A ping and capture the failed test. Restore the exact ICMP rule and re-run the test; capture recovery.

## 4. Important note about baseline testing

Terraform applies the peering connection and routes together, so the default deployment is the post-peering state. To demonstrate the pre-peering failure, conduct a controlled lab phase before creating the peer routes, or temporarily remove the route(s) before the baseline test and restore them afterward. Do not confuse an untested expected result with real evidence.

## 5. Evidence checklist

- [ ] VPC and subnet CIDRs / route table associations
- [ ] Active peering connection and ID
- [ ] Three route tables, including the peering routes
- [ ] Security group ingress rules
- [ ] Bastion SSH success
- [ ] App-A to App-B ping success
- [ ] App-B to App-A ping success
- [ ] App-B SSH through bastion
- [ ] Both private instances have no public IPv4
- [ ] Controlled ICMP failure and restored success
- [ ] Cleanup evidence
