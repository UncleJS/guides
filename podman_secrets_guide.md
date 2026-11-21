# Podman Secrets - Comprehensive Technical Guide

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)


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
- [9. Common Patterns](#9-common-patterns)
  - [9.1 Multi-Secret Application](#91-multi-secret-application)
  - [9.2 Secret Rotation](#92-secret-rotation)
  - [9.3 External Secret Manager Integration](#93-external-secret-manager-integration)
- [10. Troubleshooting](#10-troubleshooting)
- [11. Performance Considerations](#11-performance-considerations)
- [12. CLI Reference](#12-cli-reference)
- [13. Secrets vs Alternatives](#13-secrets-vs-alternatives)
- [14. References](#14-references)

---
## 1. Overview

Podman secrets store sensitive data (passwords, keys, tokens) outside the container image and hand it to containers at run time. A secret is not stored in the image or in source control, and its value is not shown in the container's `podman inspect` output.

Secrets are **not encrypted by default**. The default `file` driver keeps them in a file protected only by file permissions (see [2. Architecture](#2-architecture)). Use the `pass` driver (GPG-encrypted) or a `shell` driver backed by an external secret manager if you need encryption at rest.

---


[↑ Back to TOC](#table-of-contents)

## 2. Architecture

- **Drivers:** Set with `podman secret create --driver` (or `[secrets] driver=` in `containers.conf`):
  - `file` (default): the secret resides in a read-protected file. Data is stored unencrypted (base64-encoded in JSON), protected only by file permissions
  - `pass`: the secret resides in a GPG-encrypted file managed by `pass`, see the [OpenPGP User Guide](openpgp-user-guide.md#the-pass-password-manager)
  - `shell`: custom `list`/`lookup`/`store`/`delete` scripts, for example wrapping an external vault
- **Storage location (file driver):** under the Podman graph root, `~/.local/share/containers/storage/secrets/filedriver/` (rootless) or `/var/lib/containers/storage/secrets/filedriver/` (rootful). Metadata is kept in `secrets.json` next to it
- **Mount:** When the container is created, the secret is copied into the container's runtime directory (on tmpfs under `/run`) and mounted at `/run/secrets/<name>` inside the container
- **Permissions:** Default mode is `0444` (readable by all users in the container), owned by UID/GID 0 unless `uid=`/`gid=` are set
- **Size limit:** up to 512 kB (512000 bytes) per secret

---


[↑ Back to TOC](#table-of-contents)

## 3. Basic Secret Operations


[↑ Back to TOC](#table-of-contents)

### 3.1 Create Secrets

**From stdin:**
```bash
echo 'my_password' | podman secret create db_password -
```

**From file:**
```bash
podman secret create tls_cert /path/to/cert.pem
```

**From environment variable** (the last argument is the variable name, not its value):
```bash
podman secret create --env=true api_key API_KEY
```

**Replace an existing secret** (running containers keep the old value until recreated):
```bash
printf '%s' 'new_password' | podman secret create --replace db_password -
```

**Use a different driver:**
```bash
podman secret create --driver=pass my_secret ./secret.txt.gpg
```


[↑ Back to TOC](#table-of-contents)

### 3.2 List Secrets

```bash
podman secret ls
podman secret ls --format json
```


[↑ Back to TOC](#table-of-contents)

### 3.3 Inspect Secrets

```bash
podman secret inspect db_password
podman secret inspect --format '{{.Spec.Driver.Name}}' db_password
```

By default `inspect` shows only metadata. Anyone who can run Podman as the owning user can display the value with `--showsecret`:

```bash
podman secret inspect --showsecret --format '{{.SecretData}}' db_password
```

**Note:** This means a secret is only as safe as the account that owns it.


[↑ Back to TOC](#table-of-contents)

### 3.4 Remove Secrets

```bash
podman secret rm db_password
podman secret rm secret1 secret2  # multiple
```

---


[↑ Back to TOC](#table-of-contents)

## 4. Using Secrets in Containers


[↑ Back to TOC](#table-of-contents)

### 4.1 Basic Mount

```bash
podman run --secret db_password nginx
```

Secret accessible at: `/run/secrets/db_password`


[↑ Back to TOC](#table-of-contents)

### 4.2 Custom Target Path

```bash
podman run --secret db_password,target=/app/password nginx
```


[↑ Back to TOC](#table-of-contents)

### 4.3 Custom UID/GID

```bash
podman run --secret db_password,uid=1000,gid=1000 nginx
```


[↑ Back to TOC](#table-of-contents)

### 4.4 Custom Permissions

```bash
podman run --secret db_password,uid=101,gid=101,mode=0400 nginx
```

The default mode is `0444`. Combine `mode=0400` with a matching `uid=` so only the application user can read the file.


[↑ Back to TOC](#table-of-contents)

### 4.5 Environment Variable Type

```bash
podman run --secret db_password,type=env,target=DB_PASS nginx
```

Injects as environment variable `DB_PASS` instead of file

---


[↑ Back to TOC](#table-of-contents)

## 5. Secrets in Pods

`podman pod create` has no `--secret` option. Secrets are attached per container, so pass `--secret` to each container in the pod that needs it:

```bash
podman pod create --name mypod
podman run -d --pod mypod --name app1 --secret db_password nginx
podman run -d --pod mypod --name app2 --secret db_password redis
```

Both containers then read `/run/secrets/db_password`. Containers in the pod without `--secret` do not see it.

---


[↑ Back to TOC](#table-of-contents)

## 6. Kubernetes YAML Integration

`podman kube play` supports Kubernetes `Secret` objects. A `Secret` in the YAML is stored as a Podman secret and can be used as a volume or as `env` (`secretKeyRef`). With a volume, each key (`username`, `password`) becomes a file under `mountPath`:

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
podman kube play deployment.yaml
podman kube down deployment.yaml   # tear down
```

**Note:** Values under `data:` are only base64-encoded, not encrypted. Do not commit such YAML with real credentials.

---


[↑ Back to TOC](#table-of-contents)

## 7. Systemd Integration


[↑ Back to TOC](#table-of-contents)

### 7.1 Container Unit File

Save a Quadlet unit as `~/.config/containers/systemd/webapp.container` (rootless). The secrets must already exist (`podman secret create`).

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


[↑ Back to TOC](#table-of-contents)

### 7.2 Activation

Quadlet generates `webapp.service` from `webapp.container` at `daemon-reload`. Generated units cannot be enabled with `systemctl enable`; the `[Install]` section (`WantedBy=default.target`) is what starts them at boot (or at login for rootless).

```bash
systemctl --user daemon-reload
systemctl --user start webapp.service
systemctl --user status webapp.service
```

To start rootless services at boot without logging in, enable lingering once: `loginctl enable-linger $USER`. To debug the generated unit: `/usr/libexec/podman/quadlet -dryrun -user`.

---


[↑ Back to TOC](#table-of-contents)

## 8. Security Best Practices

| Practice | Implementation |
|----------|----------------|
| **Principle of Least Privilege** | Default mode is 0444; set `mode=0400` plus the container user's `uid`/`gid` |
| **Encryption at Rest** | The `file` driver is unencrypted; use `pass` or a `shell` driver for an external vault when needed |
| **Rotation Strategy** | `podman secret create --replace`, then recreate the containers. Automate with systemd timers or Ansible |
| **Avoid Logs and History** | Don't put secrets on the command line; read from a file, stdin, or `--env=true` |
| **Rootless by Default** | Use rootless Podman; secrets live in the user's own storage (`~/.local/share/containers/storage/secrets/`) |
| **Protect the Account** | Anyone who can run Podman as the owner can read values with `inspect --showsecret` |
| **SELinux Enforcement** | Keep SELinux enforcing (`getenforce`); container processes run as `container_t` |
| **Prefer Files over Env** | `type=env` values are visible in `/proc/<pid>/environ` and to child processes |

---


[↑ Back to TOC](#table-of-contents)

## 9. Common Patterns


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

### 9.2 Secret Rotation

```bash
#!/bin/bash
set -euo pipefail
openssl rand -base64 32 | podman secret create --replace db_password -
# Existing containers keep the old value; recreate them
podman rm -f webapp
podman run -d --name webapp --secret db_password myapp:latest
```

With Quadlet, `systemctl --user restart webapp.service` recreates the container and picks up the new value.


[↑ Back to TOC](#table-of-contents)

### 9.3 External Secret Manager Integration

**Vault integration example** (the value is copied into the file driver, so it is stored unencrypted locally):

```bash
vault kv get -field=password secret/db | \
  podman secret create db_password -
```

---


[↑ Back to TOC](#table-of-contents)

## 10. Troubleshooting

| Issue | Solution |
|-------|----------|
| **Secret not found in container** | Verify: `podman inspect --format '{{json .Config.Secrets}}' <container>` and `podman secret exists <name>` |
| **Permission denied reading secret** | Check `uid`/`gid` match the container user, adjust `mode` |
| **Container still sees old value** | Secrets are copied at container creation; recreate the container after `--replace` |
| **SELinux denials** | `sudo ausearch -m AVC -ts recent` to see denials; don't disable SELinux |
| **Unexpected trailing newline** | `echo` adds `\n`; use `printf '%s'` when piping the value |

---


[↑ Back to TOC](#table-of-contents)

## 11. Performance Considerations

- **Memory:** The in-container copy lives on tmpfs, so it uses RAM; the overhead is minimal
- **Size limit:** 512 kB per secret; use volumes or an external key manager for larger data
- **Concurrent access:** Each container gets its own copy, so no locking is needed
- **Startup time:** Negligible impact for typical secrets

---


[↑ Back to TOC](#table-of-contents)

## 12. CLI Reference

| Command | Description |
|---------|-------------|
| `podman secret create NAME FILE` | Create secret from file or stdin (-) |
| `podman secret ls` | List all secrets |
| `podman secret create --replace NAME -` | Replace an existing secret |
| `podman secret inspect NAME` | Display metadata |
| `podman secret inspect --showsecret NAME` | Display metadata and the secret value |
| `podman secret exists NAME` | Exit 0 if the secret exists |
| `podman secret rm NAME` | Remove secret |
| `podman run --secret NAME` | Mount secret in container |
| `--secret NAME,target=PATH` | Custom mount path |
| `--secret NAME,type=env` | Inject as environment variable |
| `--secret NAME,uid=N,gid=N` | Set ownership |
| `--secret NAME,mode=OCTAL` | Set permissions (default 0444) |

---


[↑ Back to TOC](#table-of-contents)

## 13. Secrets vs Alternatives

| Method | Security | Persistence | Use Case |
|--------|----------|-------------|----------|
| **Secrets** | Medium-High (not in image or inspect; unencrypted with `file` driver) | Stored until removed; tmpfs copy per container | Passwords, keys, tokens |
| **ENV vars** | Low | Process lifetime | Config, non-sensitive |
| **Bind mounts** | Medium | Persistent disk | Config files, certs |
| **Volumes** | Low | Persistent | Application data |

---


[↑ Back to TOC](#table-of-contents)

## 14. References

- Man pages: `podman-secret(1)`, `podman-secret-create(1)`, `podman-secret-inspect(1)`, `podman-run(1)`, `podman-kube-play(1)`, `podman-systemd.unit(5)`
- Podman docs: <https://docs.podman.io/en/latest/markdown/podman-secret-create.1.html>
- Quadlet spec: <https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html>
- RHEL documentation: "Building, running, and managing containers" (Red Hat Customer Portal)

[↑ Back to TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
