# Project Evidence

This directory contains sanitized screenshots captured during implementation and testing.

## Evidence Checklist

Screenshots should demonstrate:

1. EC2 instance running successfully
2. Ubuntu and `t3.micro` configuration
3. Encrypted EBS volume
4. Security Group with:
   - HTTP TCP/80 publicly accessible
   - SSH TCP/22 restricted to administrator IP
5. Nginx service running
6. Public custom status page
7. Nginx service failure test
8. Security Group failure test
9. Successful recovery
10. Nginx persistence after reboot

## Security

Screenshots published in this repository must not expose:

- Private SSH keys
- AWS credentials
- Passwords
- Access keys
- Account identifiers where unnecessary
- Administrator public IP addresses where unnecessary

Sensitive information should be cropped or redacted before publication.