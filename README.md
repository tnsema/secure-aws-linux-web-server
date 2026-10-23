# Secure AWS Linux Web Server

A hands-on AWS cloud project focused on deploying and securing a Linux web server while practising cloud security, Linux administration, troubleshooting, and operational recovery.

The goal was not simply to make a webpage publicly accessible, but to understand the infrastructure and security controls behind it.

## Architecture

```text
                        Internet
                           |
                       HTTP :80
                           |
                           v
                  +------------------+
                  | Security Group   |
                  |------------------|
                  | HTTP :80         |
                  | 0.0.0.0/0       |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  | Amazon EC2       |
                  | t3.micro         |
                  | Ubuntu 26.04 LTS |
                  | Nginx            |
                  +------------------+

Admin Workstation
       |
       | SSH :22
       | Source: Admin Public IP /32
       |
       +-----------> Security Group -----------> EC2
```

A more detailed architecture description is available in [`architecture/`](architecture/).

## Technologies

- Amazon EC2
- Amazon EBS
- AWS Security Groups
- Ubuntu Server 26.04 LTS
- Nginx
- SSH
- systemd / systemctl
- Git & GitHub

## Infrastructure

| Component | Configuration |
|---|---|
| Cloud Provider | AWS |
| Compute | Amazon EC2 |
| Instance Type | t3.micro |
| Operating System | Ubuntu Server 26.04 LTS |
| Web Server | Nginx |
| Storage | 8 GiB gp3 EBS |
| Storage Encryption | Enabled |
| Public Web Access | HTTP TCP/80 |
| Administrative Access | SSH TCP/22 |
| SSH Source | Restricted to administrator public IP `/32` |

More configuration details are documented in [`infrastructure/`](infrastructure/).

## Security Controls

### Root account protection

The AWS root account was protected with MFA and reserved for account-level operations rather than routine infrastructure administration.

### Restricted SSH

SSH access is not exposed to the entire internet.

```text
TCP 22 -> Administrator Public IP /32
```

This limits SSH connection attempts at the Security Group layer.

### Public HTTP

The web server must be reachable publicly, therefore HTTP is allowed from:

```text
TCP 80 -> 0.0.0.0/0
```

Only the service that needs public access is exposed publicly.

### Key-based authentication

The EC2 instance uses an SSH key pair rather than password-based SSH authentication.

The private key is stored locally and is never committed to this repository.

### Encrypted storage

The EC2 root EBS volume uses encryption at rest.

## Implementation

The EC2 instance was provisioned using a Free Tier eligible Ubuntu Server image and instance type.

After connecting through SSH, the package index was updated:

```bash
sudo apt update
```

Nginx was installed:

```bash
sudo apt install nginx -y
```

The service was verified:

```bash
sudo systemctl status nginx
```

Nginx was also configured to start automatically during system boot.

```bash
sudo systemctl is-enabled nginx
```

The default page was replaced with a custom operational status page.

## Validation

The server was tested externally through its public IPv4 address.

The deployed page returned successfully over HTTP and displayed:

```text
Secure AWS Linux Web Server

Status: Operational

Platform: AWS EC2 | Ubuntu | Nginx
```

A full EC2 reboot was also performed.

After the instance returned online, Nginx started automatically and the website became available again without manually starting the service.

This verified service persistence.

## Break-Fix Testing

Two deliberate failures were introduced to practise troubleshooting.

### Scenario 1 — Nginx service stopped

Nginx was intentionally stopped:

```bash
sudo systemctl stop nginx
```

The public website returned:

```text
ERR_CONNECTION_REFUSED
```

Investigation included:

```bash
curl -I http://localhost
sudo systemctl status nginx
```

The service was recovered with:

```bash
sudo systemctl restart nginx
```

The website returned to normal operation.

### Scenario 2 — HTTP blocked by Security Group

The Security Group HTTP TCP/80 inbound rule was temporarily removed.

The public website became unreachable and eventually returned:

```text
ERR_CONNECTION_TIMED_OUT
```

However:

```bash
curl -I http://localhost
```

continued returning a successful HTTP response.

This demonstrated that Nginx itself was healthy and that the failure existed at the network access layer.

Restoring the HTTP Security Group rule restored public access.

## Troubleshooting Encountered

### SSH private key permissions

The first SSH connection failed because Windows OpenSSH considered the downloaded private key permissions too permissive.

The key ACL was restricted so that only the current Windows user could read the private key.

After correcting the permissions, SSH authentication succeeded.

This reinforced an important security principle:

> A private key must not only remain secret; access to the key file itself must also be restricted.

### Human access design

The initial project plan assumed that a long-lived IAM administrator user would be used for everyday human administration.

During implementation, the current AWS account experience recommended its newer human/team access model instead.

Rather than creating unnecessary long-term credentials or access keys, the implementation was adjusted based on the environment encountered.

This became an example of validating architecture decisions against the actual cloud environment instead of blindly following the original plan.

## Monitoring and Logs

Nginx access logs were inspected using:

```bash
sudo tail -n 10 /var/log/nginx/access.log
```

Error logs were inspected using:

```bash
sudo tail -n 10 /var/log/nginx/error.log
```

These logs provide evidence for troubleshooting HTTP requests and server-side errors.

## Incident Recovery Runbook

A reusable Nginx outage recovery procedure is documented here:

[`docs/nginx-outage-runbook.md`](docs/nginx-outage-runbook.md)

## Evidence

Sanitized implementation screenshots are available in:

[`evidence/`](evidence/)

Sensitive information such as private SSH keys and unnecessary account/network