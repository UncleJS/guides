# Podman Secrets - Comprehensive Technical Guide

**RHEL 10 | Enterprise Container Security**

---


## Table of Contents

- [1. Overview](#1-overview)
- [2. Architecture](#2-architecture)
- [3. Basic Secret Operations](#3-basic-secret-operations)
  - [3.1 Create Secrets](#31-create-secrets)
  - [3.2 List Secrets](#32-list-secrets)
  - [3.3 Inspect Secrets](#33-inspect-secrets)
  - [3.4 Remove Secrets](#34-remove-secrets)
- [4. Using Secrets in Containers](#4-using-secrets-in-containers)
  - [4.1 Basic Mount](#41-basic-mount)
  - [4.2 Custom Target Path](#42-custom-target-path)
  - [4.3 Custom UID/GID](#43-custom-uidgid)
  - [4.4 Custom Permissions](#44-custom-permissions)
  - [4.5 Environment Variable Type](#45-environment-variable-type)
- [5. Secrets in Pods](#5-secrets-in-pods)
- [6. Kubernetes YAML Integration](#6-kubernetes-yaml-integration)
- [7. Systemd Integration](#7-systemd-integration)
  - [7.1 Container Unit File](#71-container-unit-file)
  - [7.2 Activation](#72-activation)
- [8. Security Best Practices](#8-security-best-practices)
- [9. Advanced Patterns](#9-advanced-patterns)
  - [9.1 Multi-Secret Application](#91-multi-secret-application)
  - [9.2 Secret Rotation Script](#92-secret-rotation-script)
  - [9.3 External Secret Manager Integration](#93-external-secret-manager-integration)
- [10. Troubleshooting](#10-troubleshooting)
- [11. Performance Considerations](#11-performance-considerations)
- [12. CLI Reference](#12-cli-reference)
- [13. Secrets vs Alternatives](#13-secrets-vs-alternatives)
- [14. RHEL 10 Specific Features](#14-rhel-10-specific-features)
- [15. References](#15-references)

---
## 1. Overview

Podman secrets provide secure storage and management of sensitive data (passwords, keys, tokens) for containers. Unlike environment variables or bind mounts, secrets are stored encrypted and accessed via tmpfs, never written to disk in plaintext.

---


[↑ Goto TOC](#table-of-contents)

## 2. Architecture

- **Storage backend:** Default uses `/run/user/$UID/containers/secrets` (rootless) or `/run/containers/secrets` (rootful)
- **Encryption:** Secrets encrypted at rest using kernel keyring or filesystem permissions
- **Mount:** Injected as tmpfs at `/run/secrets/<name>` inside containers
- **Permissions:** 0400 (read-only), owned by container user

---


[↑ Goto TOC](#table-of-contents)

## 3. Basic Secret Operations


[↑ Goto TOC](#table-of-contents)

### 3.1 Create Secrets

**From stdin:**
```bash
echo 'my_password' | podman secret create db_password -
```

**From file:**
```bash
podman secret create tls_cert /path/to/cert.pem
```

**From environment variable:**
```bash
echo "$API_KEY" | podman secret create api_key -
```


[↑ Goto TOC](#table-of-contents)

### 3.2 List Secrets

```bash
podman secret ls
podman secret ls --format json
```


[↑ Goto TOC](#table-of-contents)

### 3.3 Inspect Secrets

```bash
podman secret inspect db_password
```

**Note:** Cannot retrieve secret value after creation (security feature)


[↑ Goto TOC](#table-of-contents)

### 3.4 Remove Secrets

```bash
podman secret rm db_password
podman secret rm secret1 secret2  # multiple
```

---


[↑ Goto TOC](#table-of-contents)

## 4. Using Secrets in Containers


[↑ Goto TOC](#table-of-contents)

### 4.1 Basic Mount

```bash
podman run --secret db_password nginx
```

Secret accessible at: `/run/secrets/db_password`


[↑ Goto TOC](#table-of-contents)

### 4.2 Custom Target Path

```bash
podman run --secret db_password,target=/app/password nginx
```


[↑ Goto TOC](#table-of-contents)

### 4.3 Custom UID/GID

```bash
podman run --secret db_password,uid=1000,gid=1000 nginx
```


[↑ Goto TOC](#table-of-contents)

### 4.4 Custom Permissions

```bash
podman run --secret db_password,mode=0440 nginx
```


[↑ Goto TOC](#table-of-contents)

### 4.5 Environment Variable Type

```bash
podman run --secret db_password,type=env,target=DB_PASS nginx
```

Injects as environment variable `DB_PASS` instead of file

---


[↑ Goto TOC](#table-of-contents)

## 5. Secrets in Pods

Secrets can be shared across containers in a pod:

```bash
podman pod create --name mypod --secret db_password
podman run -d --pod mypod --name app1 nginx
podman run -d --pod mypod --name app2 redis
```

Both containers access `/run/secrets/db_password`

---


[↑ Goto TOC](#table-of-contents)

## 6. Kubernetes YAML Integration

Podman supports Kubernetes-style secret definitions for `podman play kube`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
---
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: db-creds
      mountPath: /run/secrets
      readOnly: true
  volumes:
  - name: db-creds
    secret:
      secretName: db-credentials
```

```bash
podman play kube deployment.yaml
```

---


[↑ Goto TOC](#table-of-contents)

## 7. Systemd Integration


[↑ Goto TOC](#table-of-contents)

### 7.1 Container Unit File

```ini
[Unit]
Description=Web Application
After=network-online.target

[Container]
Image=nginx:latest
Secret=db_password
Secret=api_key,target=/etc/api.key,mode=0400
PublishPort=8080:80

[Service]
Restart=always

[Install]
WantedBy=default.target
```


[↑ Goto TOC](#table-of-contents)

### 7.2 Activation

```bash
systemctl --user daemon-reload
systemctl --user enable --now webapp.container
```

---


[↑ Goto TOC](#table-of-contents)

## 8. Security Best Practices

| Practice | Implementation |
|----------|----------------|
| **Principle of Least Privilege** | Use mode=0400 (read-only), set specific uid/gid per container |
| **Rotation Strategy** | Delete old secret, create new, restart containers. Automate with cron/Ansible |
| **Avoid Logs** | Never echo secrets, use process substitution, redirect stdin only |
| **Rootless by Default** | Use rootless Podman (user namespace), secrets stored in user-specific location |
| **SELinux Enforcement** | Keep SELinux enforcing, verify contexts: `ls -Z /run/containers/secrets` |
| **Audit Access** | Monitor with auditd, `journalctl -u container-*`, track secret usage |

---


[↑ Goto TOC](#table-of-contents)

## 9. Advanced Patterns


[↑ Goto TOC](#table-of-contents)

### 9.1 Multi-Secret Application

```bash
podman run -d \
  --name webapp \
  --secret db_password,target=/run/secrets/db/password \
  --secret db_user,target=/run/secrets/db/user \
  --secret api_key,type=env,target=API_KEY \
  --secret tls_cert,target=/etc/ssl/certs/app.crt \
  --secret tls_key,target=/etc/ssl/private/app.key,mode=0400 \
  myapp:latest
```


[↑ Goto TOC](#table-of-contents)

### 9.2 Secret Rotation Script

```bash
#!/bin/bash
NEW_PASS=$(openssl rand -base64 32)
echo "$NEW_PASS" | podman secret create db_password_new -
podman stop webapp
podman rm webapp
podman secret rm db_password
echo "$NEW_PASS" | podman secret create db_password -
podman secret rm db_password_new
podman run -d --name webapp --secret db_password myapp:latest
```


[↑ Goto TOC](#table-of-contents)

### 9.3 External Secret Manager Integration

**Vault integration example:**

```bash
vault kv get -field=password secret/db | \
  podman secret create db_password -
```

---


[↑ Goto TOC](#table-of-contents)

## 10. Troubleshooting

| Issue | Solution |
|-------|----------|
| **Secret not found in container** | Verify: `podman inspect <container> \| grep -A5 Secrets` |
| **Permission denied reading secret** | Check uid/gid match container user, adjust mode/ownership |
| **Cannot remove secret (in use)** | Stop/remove containers: `podman ps -a --filter volume=<secret>` |
| **SELinux denials** | `audit2allow -a \| grep secrets`, `restorecon -Rv /run/containers` |
| **Empty secret content** | Check stdin pipe, use `-` correctly, verify no whitespace/newlines |

---


[↑ Goto TOC](#table-of-contents)

## 11. Performance Considerations

- **Tmpfs overhead:** Minimal, mounted in-memory per container
- **Large secrets:** Keep under 1MB, consider external key management for larger data
- **Concurrent access:** No locking required, each container gets independent tmpfs mount
- **Startup time:** Negligible impact (<10ms per secret)

---


[↑ Goto TOC](#table-of-contents)

## 12. CLI Reference

| Command | Description |
|---------|-------------|
| `podman secret create NAME FILE` | Create secret from file or stdin (-) |
| `podman secret ls` | List all secrets |
| `podman secret inspect NAME` | Display metadata (not value) |
| `podman secret rm NAME` | Remove secret |
| `podman run --secret NAME` | Mount secret in container |
| `--secret NAME,target=PATH` | Custom mount path |
| `--secret NAME,type=env` | Inject as environment variable |
| `--secret NAME,uid=N,gid=N` | Set ownership |
| `--secret NAME,mode=OCTAL` | Set permissions (default 0400) |

---


[↑ Goto TOC](#table-of-contents)

## 13. Secrets vs Alternatives

| Method | Security | Persistence | Use Case |
|--------|----------|-------------|----------|
| **Secrets** | High | Volatile (tmpfs) | Passwords, keys, tokens |
| **ENV vars** | Low | Process lifetime | Config, non-sensitive |
| **Bind mounts** | Medium | Persistent disk | Config files, certs |
| **Volumes** | Low | Persistent | Application data |

---


[↑ Goto TOC](#table-of-contents)

## 14. RHEL 10 Specific Features

- **Podman 5.x:** Enhanced secret driver support, improved passthrough security
- **SELinux:** container_secret_t context, enforcing by default
- **FIPS 140-3:** Validated crypto for secret encryption when FIPS mode enabled
- **Systemd 256+:** Native Quadlet .container format with Secret= directive
- **cgroup v2:** Unified hierarchy, improved resource limits around secret tmpfs

---


[↑ Goto TOC](#table-of-contents)

## 15. References

- Official docs: `podman-secret(1)`, `podman-run(1)`, `podman-pod-create(1)`
- RHEL 10 Security Guide: containers.podman.io/security
- SELinux policy: selinux-policy-devel, container_secrets.te
- Quadlet spec: docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html

[↑ Goto TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
