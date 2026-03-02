# Ubuntu Server Baseline Summary

This document provides a summary of the roles included in this playbook, which is designed to harden Ubuntu servers according to CIS benchmarks.

## Roles

### `common`
This role performs initial, common security hardening tasks that don't fit into other categories. It includes:
- Disabling uncommon and potentially insecure filesystem modules (cramfs, freevxfs, etc.).
- Installing and enabling AppArmor for mandatory access control.
- Installing `sudo` and configuring it to use a separate log file and require a `pty`.
- Installing and configuring AIDE for file integrity monitoring, including a cron job for daily checks.

### `users`
This role focuses on user and group management security. It includes:
- Enforcing strong password policies (length, complexity) using `pam_pwquality`.
- Configuring account lockout after multiple failed login attempts using `pam_faillock`.
- Setting password expiration policies (max days, min days, warning age).
- Disabling the root account's password to prevent direct root login.
- Setting a restrictive default umask (027) for new files.

### `ssh`
This role hardens the SSH server (`sshd`). It includes:
- Disabling password-based authentication, requiring SSH key-based authentication.
- Disabling direct root login via SSH.
- Configuring timeouts to prevent idle sessions.
- Enabling strict mode and disabling empty passwords.
- Enforcing the use of strong cryptographic ciphers, MACs, and key exchange algorithms.

### `firewall`
This role configures a host-based firewall using `ufw` (Uncomplicated Firewall). It includes:
- Installing `ufw`.
- Setting a default-deny policy for all incoming, outgoing, and routed traffic.
- Allowing traffic only on the standard SSH port (22/tcp).
- Enabling the firewall service.

### `packages`
This role reduces the system's attack surface by removing unnecessary packages. This includes:
- X11 server components.
- Common server services that are not needed on a dedicated appliance (e.g., Avahi, CUPS, DHCP, LDAP, NFS, DNS, FTP, HTTP, Samba).

### `unattended_upgrades`
This role ensures the system is kept up-to-date with the latest security patches. It includes:
- Installing the `unattended-upgrades` package.
- Configuring `apt` to automatically download and install security updates.
- Enabling and starting the unattended upgrades service.

### `kernel`
This role hardens the kernel's networking stack by configuring `sysctl` parameters. It includes:
- Disabling IP forwarding.
- Ignoring ICMP redirects and broadcasts.
- Enabling TCP SYN cookies to mitigate SYN flood attacks.
- Enabling reverse path filtering to prevent IP spoofing.
- Logging suspicious packets.

### `auditd`
This role configures the Linux Audit Daemon to log security-relevant events. It includes:
- Installing `auditd`.
- Configuring log rotation and size.
- Adding a set of CIS-compliant audit rules to log events related to:
  - Identity and access (changes to `/etc/passwd`, `/etc/group`, etc.).
  - System calls and module loading.
  - Time changes.
- Making the audit configuration immutable to prevent tampering.
