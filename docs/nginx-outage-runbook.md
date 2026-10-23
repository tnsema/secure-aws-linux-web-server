# Nginx Outage Recovery Runbook

## Purpose

This runbook provides a repeatable procedure for diagnosing and recovering the public Nginx web service running on the EC2 instance.

## Symptom

The public web page does not load, times out, refuses the connection, or returns an HTTP 5xx response.

## 1. Verify EC2

Confirm in the AWS Console that the EC2 instance is:

```text
Running
```

and that its status checks are passing.

## 2. Verify Network Access

Check the EC2 Security Group.

Expected inbound rules:

```text
SSH   TCP 22   Administrator IP /32
HTTP  TCP 80   0.0.0.0/0
```

If HTTP port 80 is missing, external clients will not be able to reach Nginx.

## 3. Connect to the Server

```bash
ssh -i "secure-web-server-key.pem" ubuntu@<EC2-PUBLIC-DNS>
```

## 4. Test Nginx Locally

```bash
curl -I http://localhost
```

If this succeeds while the public website fails, investigate the Security Group or other network controls.

## 5. Inspect Service Status

```bash
sudo systemctl status nginx
```

If Nginx is inactive or failed, continue with log inspection.

## 6. Inspect Logs

```bash
sudo tail -n 50 /var/log/nginx/error.log
```

Access requests can also be inspected with:

```bash
sudo tail -n 50 /var/log/nginx/access.log
```

## 7. Recover the Service

```bash
sudo systemctl restart nginx
```

Verify:

```bash
sudo systemctl status nginx
curl -I http://localhost
```

## 8. Verify Externally

Open the EC2 public address from a browser and confirm that the operational status page loads.

## Diagnostic Pattern

```text
Public fails + localhost fails
        ↓
Investigate Nginx/service

Public fails + localhost works
        ↓
Investigate Security Group/network

Public works
        ↓
Service operational
```

## Recovery Complete

The incident is considered recovered when:

- Nginx reports `active (running)`
- `curl` returns a successful HTTP response
- The public status page is reachable