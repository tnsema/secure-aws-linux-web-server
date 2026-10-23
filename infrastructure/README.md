# Infrastructure Configuration

This directory documents the infrastructure configuration used for the Secure AWS Linux Web Server project.

## EC2 Instance

| Setting | Configuration |
|---|---|
| Cloud Provider | AWS |
| Service | Amazon EC2 |
| Operating System | Ubuntu Server 26.04 LTS |
| Architecture | x86_64 |
| Instance Type | t3.micro |
| Web Server | Nginx |
| Root Storage | 8 GiB gp3 EBS |
| EBS Encryption | Enabled |

## Network Access

The EC2 instance uses an AWS Security Group as its network firewall.

### Inbound Rules

| Service | Protocol | Port | Source | Purpose |
|---|---|---:|---|---|
| SSH | TCP | 22 | Administrator IP `/32` | Secure remote administration |
| HTTP | TCP | 80 | `0.0.0.0/0` | Public web access |

SSH is deliberately restricted to a single administrator public IP instead of being exposed to the entire internet.

## SSH Authentication

The server uses key-based SSH authentication.

```bash
ssh -i "secure-web-server-key.pem" ubuntu@<EC2-PUBLIC-DNS>
```

The private `.pem` file is not stored in this repository.

## Nginx

Installation:

```bash
sudo apt update
sudo apt install nginx -y
```

Service verification:

```bash
sudo systemctl status nginx
```

Boot persistence verification:

```bash
sudo systemctl is-enabled nginx
```

Local HTTP verification:

```bash
curl -I http://localhost
```

## Logs

Access log:

```bash
sudo tail -n 10 /var/log/nginx/access.log
```

Error log:

```bash
sudo tail -n 10 /var/log/nginx/error.log
```

## Security Decisions

The implementation uses:

- MFA protection for the AWS root account
- Restricted SSH access
- Key-based SSH authentication
- Encrypted EBS storage
- Minimal publicly exposed ports
- No private keys or AWS credentials stored in Git
- Service persistence through systemd