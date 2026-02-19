# Podman Secrets - Comprehensive Technical Guide

**RHEL 10 | Enterprise Container Security**

---

## 1. Overview

Podman secrets provide secure storage and management of sensitive data (passwords, keys, tokens) for containers. Unlike environment variables or bind mounts, secrets are stored encrypted and accessed via tmpfs, never written to disk in plaintext.

---

## 2. Architecture

- **Storage backend:** Default uses `/run/user/$UID/containers/secrets` (rootless) or `/run/containers/secrets` (rootful)
- **Encryption:** Secrets encrypted at rest using kernel keyring or filesystem permissions
- **Mount:** Injected as tmpfs at `/run/secrets/<name>` inside containers
- **Permissions:** 0400 (read-only), owned by container user

---

## 3. Basic Secret Operations

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

### 3.2 List Secrets

```bash
podman secret ls
podman secret ls --format json
```

### 3.3 Inspect Secrets

```bash
podman secret inspect db_password
```

**Note:** Cannot retrieve secret value after creation (security feature)

### 3.4 Remove Secrets

```bash
podman secret rm db_password
podman secret rm secret1 secret2  # multiple
```

---

## 4. Using Secrets in Containers

### 4.1 Basic Mount

```bash
podman run --secret db_password nginx
```

Secret accessible at: `/run/secrets/db_password`

### 4.2 Custom Target Path

```bash
podman run --secret db_password,target=/app/password nginx
```

### 4.3 Custom UID/GID

```bash
podman run --secret db_password,uid=1000,gid=1000 nginx
```

### 4.4 Custom Permissions

```bash
podman run --secret db_password,mode=0440 nginx
```

### 4.5 Environment Variable Type

```bash
podman run --secret db_password,type=env,target=DB_PASS nginx
```

Injects as environment variable `DB_PASS` instead of file

---

## 5. Secrets in Pods

Secrets can be shared across containers in a pod:

```bash
podman pod create --name mypod --secret db_password
podman run -d --pod mypod --name app1 nginx
podman run -d --pod mypod --name app2 redis
```

Both containers access `/run/secrets/db_password`

---

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

## 7. Systemd Integration

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

### 7.2 Activation

```bash
systemctl --user daemon-reload
systemctl --user enable --now webapp.container
```

---

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

## 9. Advanced Patterns

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

### 9.3 External Secret Manager Integration

**Vault integration example:**

```bash
vault kv get -field=password secret/db | \
  podman secret create db_password -
```

---

## 10. Troubleshooting

| Issue | Solution |
|-------|----------|
| **Secret not found in container** | Verify: `podman inspect <container> \| grep -A5 Secrets` |
| **Permission denied reading secret** | Check uid/gid match container user, adjust mode/ownership |
| **Cannot remove secret (in use)** | Stop/remove containers: `podman ps -a --filter volume=<secret>` |
| **SELinux denials** | `audit2allow -a \| grep secrets`, `restorecon -Rv /run/containers` |
| **Empty secret content** | Check stdin pipe, use `-` correctly, verify no whitespace/newlines |

---

## 11. Performance Considerations

- **Tmpfs overhead:** Minimal, mounted in-memory per container
- **Large secrets:** Keep under 1MB, consider external key management for larger data
- **Concurrent access:** No locking required, each container gets independent tmpfs mount
- **Startup time:** Negligible impact (<10ms per secret)

---

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

## 13. Secrets vs Alternatives

| Method | Security | Persistence | Use Case |
|--------|----------|-------------|----------|
| **Secrets** | High | Volatile (tmpfs) | Passwords, keys, tokens |
| **ENV vars** | Low | Process lifetime | Config, non-sensitive |
| **Bind mounts** | Medium | Persistent disk | Config files, certs |
| **Volumes** | Low | Persistent | Application data |

---

## 14. RHEL 10 Specific Features

- **Podman 5.x:** Enhanced secret driver support, improved passthrough security
- **SELinux:** container_secret_t context, enforcing by default
- **FIPS 140-3:** Validated crypto for secret encryption when FIPS mode enabled
- **Systemd 256+:** Native Quadlet .container format with Secret= directive
- **cgroup v2:** Unified hierarchy, improved resource limits around secret tmpfs

---

## 15. References

- Official docs: `podman-secret(1)`, `podman-run(1)`, `podman-pod-create(1)`
- RHEL 10 Security Guide: containers.podman.io/security
- SELinux policy: selinux-policy-devel, container_secrets.te
- Quadlet spec: docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html
