# Architecture

## Overview

The project uses a simple AWS architecture designed around controlled administrative access and public HTTP availability.

```text
                         INTERNET
                            |
                            | HTTP TCP/80
                            | 0.0.0.0/0
                            v
                 +-----------------------+
                 | AWS Security Group    |
                 | secure-web-server-sg  |
                 |-----------------------|
                 | HTTP 80: Public       |
                 | SSH 22: Admin IP /32  |
                 +-----------+-----------+
                             |
                             v
                 +-----------------------+
                 | Amazon EC2            |
                 |-----------------------|
                 | t3.micro              |
                 | Ubuntu Server 26.04   |
                 | Nginx                 |
                 |                       |
                 | 8 GiB gp3 EBS         |
                 | Encrypted             |
                 +-----------------------+
                             ^
                             |
                             | SSH TCP/22
                             |
                    +--------+--------+
                    | Admin Workstation|
                    | SSH Private Key  |
                    +-----------------+
```

## Request Flow

Public web traffic follows:

```text
Browser
   ↓
Public EC2 IPv4 Address
   ↓
Security Group — TCP/80
   ↓
EC2
   ↓
Nginx
   ↓
Static Status Page
```

Administrative traffic follows:

```text
Administrator Workstation
   ↓
SSH private key
   ↓
TCP/22
   ↓
Security Group
   ↓
Source IP /32 validation
   ↓
EC2 Ubuntu Server
```

## Security Boundary

The Security Group acts as the primary network access control around the EC2 instance.

Only two inbound services are exposed:

| Port | Service | Source |
|---:|---|---|
| 22 | SSH | Administrator public IP `/32` |
| 80 | HTTP | `0.0.0.0/0` |

Port 22 is restricted because SSH provides administrative access to the server.

Port 80 is publicly accessible because the purpose of the workload is to serve a public web page.

## Storage

The EC2 instance uses an:

```text
8 GiB gp3 encrypted EBS volume
```

Encryption protects the volume data at rest.

## Availability and Persistence

Nginx is managed through `systemd`.

The service is enabled at boot so that restarting the EC2 operating system does not require Nginx to be started manually.

Persistence was verified by rebooting the EC2 instance and confirming that Nginx returned automatically.