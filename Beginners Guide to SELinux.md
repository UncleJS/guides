# Beginner’s Guide to SELinux

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

## Table of Contents

- [Introduction to SELinux](#introduction-to-selinux)
  - [Why Use SELinux?](#why-use-selinux)
- [SELinux Modes](#selinux-modes)
- [Key Concepts in SELinux](#key-concepts-in-selinux)
  - [Contexts (Labels)](#contexts-labels)
  - [Policies](#policies)
  - [Booleans](#booleans)
- [Managing SELinux Contexts](#managing-selinux-contexts)
  - [Viewing and Changing Contexts](#viewing-and-changing-contexts)
  - [Ports](#ports)
- [Troubleshooting SELinux Issues](#troubleshooting-selinux-issues)
- [Common Use Cases](#common-use-cases)
  - [Web Server (Apache)](#web-server-apache)
  - [Database (MySQL/MariaDB)](#database-mysqlmariadb)
  - [Custom Applications](#custom-applications)
- [Best Practices for Beginners](#best-practices-for-beginners)
- [Advanced Topics (For Later)](#advanced-topics-for-later)

---
## Introduction to SELinux

SELinux (Security-Enhanced Linux) is a security module integrated into the Linux kernel that provides a mechanism for supporting access control security policies. It was originally developed by the United States National Security Agency (NSA) and has been part of the mainline Linux kernel since version 2.6.

Unlike traditional Discretionary Access Control (DAC) systems (like standard Unix permissions with users, groups, and modes such as rwxr-xr-x), SELinux implements Mandatory Access Control (MAC). In MAC, the operating system enforces access policies that cannot be overridden by users or processes, even if they run as root. This adds an extra layer of security, preventing unauthorized actions and containing potential breaches.

SELinux is commonly used in enterprise environments, servers, and distributions like Red Hat Enterprise Linux (RHEL), CentOS, Fedora, and even some Android systems. It's designed to protect against exploits, misconfigurations, and insider threats by confining processes to only the resources they need.


[↑ Back to TOC](#table-of-contents)

### Why Use SELinux?
- **Enhanced Security**: Limits damage from compromised applications.
- **Compliance**: Meets standards like PCI DSS, HIPAA, or government requirements.
- **Fine-Grained Control**: Policies can be tailored to specific needs.
- **Auditability**: Logs access denials for troubleshooting and forensics.

However, it can be complex for beginners, often leading to frustration with "permission denied" errors that aren't related to standard file permissions.


[↑ Back to TOC](#table-of-contents)

## SELinux Modes

SELinux operates in one of three modes:

1. **Enforcing**: SELinux security policy is enforced. If a policy violation occurs, access is denied, and the action is logged.
2. **Permissive**: Violations are logged but not enforced. This is useful for testing and troubleshooting without blocking operations.
3. **Disabled**: SELinux is completely turned off. Not recommended; prefer Permissive when you need to stop enforcement.

To check the current mode, use:
```
sestatus
```
Or for a quick check:
```
getenforce
```

To temporarily switch modes (until reboot):
```
setenforce 0  # Permissive
setenforce 1  # Enforcing
```

For permanent changes between Enforcing and Permissive, edit `/etc/selinux/config` (on modern RHEL/Fedora this is canonical; `/etc/sysconfig/selinux` is a legacy symlink to it):
```
SELINUX=enforcing
# or SELINUX=permissive
```
Then reboot the system.

**Disabling SELinux on RHEL 9+ and current Fedora**: `SELINUX=disabled` in `/etc/selinux/config` no longer fully disables SELinux (the kernel still initializes it; policy is just not loaded). To really disable it, use the kernel command line instead:
```
sudo grubby --update-kernel ALL --args selinux=0
# to undo: sudo grubby --update-kernel ALL --remove-args selinux
```
Re-enabling later requires a full relabel (`sudo touch /.autorelabel` and reboot). In almost all cases, Permissive mode is the better choice.

**Tip**: Start in Permissive mode when learning to avoid disrupting your system.


[↑ Back to TOC](#table-of-contents)

## Key Concepts in SELinux

Three ideas cover most day-to-day SELinux work: **contexts** (labels on every process and object), **policies** (the rules that compare those labels), and **booleans** (on/off switches for optional parts of the policy).


[↑ Back to TOC](#table-of-contents)

### Contexts (Labels)
Every file, process, port, and user in SELinux has a **security context** (also called a label). This is a string that defines the security attributes, typically in the format:
```
user:role:type:level
```
- **User**: SELinux user (e.g., `system_u` for system processes).
- **Role**: Role-based access (e.g., `object_r` for objects).
- **Type**: The most important part for type enforcement (e.g., `httpd_sys_content_t` for web server content).
- **Level**: For Multi-Level Security (MLS) or Multi-Category Security (MCS), often optional in basic setups.

View contexts with:
- Files: `ls -Z`
- Processes: `ps -eZ`
- Ports: `semanage port -l`

Example output for a file:
```
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
```


[↑ Back to TOC](#table-of-contents)

### Policies
SELinux policies define rules for what subjects (processes) can do to objects (files, ports, etc.). Common policy types:
- **Targeted**: Default on Fedora and RHEL; confines specific services like HTTPD, while others run unconfined.
- **Minimum**: A reduced version of targeted that confines only selected processes (useful on small or embedded systems).
- **MLS**: For multi-level security (e.g., classified environments).

The old "strict" policy is obsolete; its behaviour is now achieved within the targeted policy by confining users. Check the active policy with `sestatus` (the "Loaded policy name" line).

Install policy tools if needed:
```
sudo dnf install policycoreutils-python-utils  # On RHEL/CentOS/Fedora
sudo apt install policycoreutils selinux-utils selinux-policy-dev  # On Debian/Ubuntu (SELinux support is limited here)
```


[↑ Back to TOC](#table-of-contents)

### Booleans
Booleans are toggles that allow or deny certain behaviors without rewriting policies. For example, `httpd_can_network_connect` allows Apache to connect to networks.

List all booleans:
```
getsebool -a
```

Get a specific one:
```
getsebool httpd_can_network_connect
```

Set a boolean (temporarily):
```
setsebool httpd_can_network_connect 1
```

Make it permanent:
```
setsebool -P httpd_can_network_connect 1
```


[↑ Back to TOC](#table-of-contents)

## Managing SELinux Contexts

Most SELinux problems are caused by wrong labels: a file in an unexpected location, or a service on a non-standard port. The fix is to tell the policy what label to apply, rather than to turn SELinux off.


[↑ Back to TOC](#table-of-contents)

### Viewing and Changing Contexts
View a file's context with `ls -Z /path/to/file`.

The persistent way to change a context is to add a file-context rule with `semanage fcontext`, then apply it with `restorecon`:
```
sudo semanage fcontext -a -t httpd_sys_content_t "/path/to/dir(/.*)?"
sudo restorecon -Rv /path/to/dir
```

- `semanage fcontext -a`: Adds a file context mapping to the local policy (list yours with `semanage fcontext -l -C`).
- `restorecon`: Applies the contexts the policy defines for those paths.

`chcon` changes a label directly, but only temporarily: the change is lost the next time `restorecon` runs or the file system is relabeled. Use it only for quick tests:
```
chcon -t httpd_sys_content_t /path/to/file
```


[↑ Back to TOC](#table-of-contents)

### Ports
Allow a service on a non-standard port:
```
semanage port -a -t http_port_t -p tcp 8080
```

List ports:
```
semanage port -l | grep http
```


[↑ Back to TOC](#table-of-contents)

## Troubleshooting SELinux Issues

SELinux denials are logged in `/var/log/audit/audit.log` (if auditd is running) or `/var/log/messages`.

Install tools to analyze logs:
```
sudo dnf install setroubleshoot setroubleshoot-server
```

Then, use:
```
sealert -a /var/log/audit/audit.log
```
This provides human-readable explanations and suggested fixes, like setting booleans or relabeling files.

Common issues:
- **AVC Denials**: Access Vector Cache denials indicate policy violations.
- Example log: `type=AVC msg=audit(...): avc: denied { read } for pid=... comm="httpd" name="file" ...`

To generate a custom policy module for allowing a denial:
1. Pipe the audit log to `audit2allow`:
   ```
   grep denied /var/log/audit/audit.log | audit2allow -M mymodule
   ```
2. Install the module:
   ```
   semodule -i mymodule.pp
   ```

**Warning**: Custom modules can weaken security; use them sparingly and understand the implications.


[↑ Back to TOC](#table-of-contents)

## Common Use Cases

The pattern is the same for every service: put files in the default locations when you can, label custom locations with `semanage fcontext` + `restorecon`, and enable only the booleans you need.


[↑ Back to TOC](#table-of-contents)

### Web Server (Apache)
- Ensure web content has `httpd_sys_content_t` context (default under `/var/www`; use `semanage fcontext` for other paths).
- Allow scripts: Set boolean `httpd_enable_cgi` if needed.
- For home directories: `setsebool -P httpd_enable_homedirs 1`


[↑ Back to TOC](#table-of-contents)

### Database (MySQL/MariaDB)
- Custom data directory (persistent):
  ```
  sudo semanage fcontext -a -t mysqld_db_t "/path/to/data(/.*)?"
  sudo restorecon -Rv /path/to/data
  ```
- Non-standard port: `sudo semanage port -a -t mysqld_port_t -p tcp 3307`
- Allow Apache (e.g., a PHP app) to reach the database over the network: `sudo setsebool -P httpd_can_network_connect_db 1`. There is no `mysqld_can_network_connect` boolean; list related ones with `getsebool -a | grep -E 'db|mysql'`.


[↑ Back to TOC](#table-of-contents)

### Custom Applications
- If your app needs special access, create a policy module or use `semanage` to map contexts.


[↑ Back to TOC](#table-of-contents)

## Best Practices for Beginners
- **Start Simple**: Run in Permissive mode, fix issues, then switch to Enforcing.
- **Backup Configurations**: Before changes, back up `/etc/selinux/config`.
- **Learn Incrementally**: Focus on one service at a time.
- **Resources**:
  - Red Hat SELinux Guide: Official documentation for RHEL-based systems.
  - Fedora Wiki: Practical examples.
  - Books: "SELinux by Example" for deeper dives.
- **Tools**: Use `sesearch` to query policy rules, e.g., `sesearch -A -s httpd_t -t httpd_sys_content_t`.


[↑ Back to TOC](#table-of-contents)

## Advanced Topics (For Later)
- **Policy Development**: Writing custom policies with `sepolicy`.
- **Users and Roles**: Mapping Linux users to SELinux users with `semanage login` (and managing SELinux users' roles with `semanage user`).
- **Containers**: SELinux confines Podman containers (`container_t`) on Fedora/RHEL; adjust labels or `container-selinux` booleans (`getsebool -a | grep container`) instead of disabling it.
- **Auditing**: Configuring audit rules for specific monitoring.

Remember, SELinux is powerful but requires patience. Practice on a virtual machine to avoid locking yourself out of production systems. If you encounter specific errors, tools like `sealert` are your best friend!

[↑ Back to TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
