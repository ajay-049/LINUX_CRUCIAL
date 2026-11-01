# Linux SSH Hardening & Brute-Force Mitigation

## Overview
SSH (Secure Shell) is the primary gateway for remote administration in Linux environments. By default, it operates on TCP port 22 and allows standard authentication mechanisms. In internet-facing production systems, default settings attract continuous automated port scans, brute-force attempts, and dictionary attacks. 

Hardening SSH involves migrating the service away from well-known attack vectors and enforcing kernel-level connection thresholds to secure infrastructure access.

---

## Why Is SSH Hardening Necessary?

1. **High Automated Attack Volume:** Standard port 22 is continuously probed by internet bots searching for open endpoints and default credentials.
2. **Resource & Log Exhaustion:** Thousands of failed authentication handshakes consume CPU cycles, exhaust daemon connection limits, and pollute system logs (`auth.log` / `secure`).
3. **Lockout Protection:** Uncontrolled brute-force attempts can trigger accidental account lockouts or degrade legitimate administrative connectivity.

---

## Core Components of the Setup

### 1. Custom Port Migration
Moving the SSH daemon from default port `22` to a non-standard custom port removes the server from 99% of untargeted, broad-spectrum internet port scans.

### 2. Kernel-Level Rate Limiting (iptables `recent` module)
Instead of relying solely on application-level tools, connection thresholds are enforced directly in the Linux network filter layer:
* **Hit Counter Tracking:** Monitors new connection attempts per source IP within a rolling time window (e.g., maximum 6 attempts within 30 seconds).
* **Automated TCP Reset:** Exceeded thresholds trigger an immediate kernel-level reject/drop, dropping the malicious connection before it touches the `sshd` process.

### 3. IP Priority Bypass
Static, trusted management networks (such as NOC/Office IPs and trusted jump hosts) are placed ahead of rate-limiting rules to ensure unthrottled, guaranteed access during emergency maintenance.

---

## Key Benefits

* **Zero-Noise Logs:** Drastically reduces failed login attempts and keeps security audit trails clean and readable.
* **Kernel-Level Defense:** Rejection happens in the network stack, saving server memory and CPU for production workloads.
* **Credential Stuffing Prevention:** Rate limiting makes dictionary attacks and automated password guessing practically impossible.
* **Low Maintenance:** Built purely on native Linux utilities (`sshd_config`, `iptables`, `netfilter-persistent`) without requiring third-party monitoring agents.

---

## Best Practices Checklist

- [x] Run `sshd -t` before restarting the SSH service to avoid syntax lockout.
- [x] Keep an active SSH session alive while verifying access through a second terminal.
- [x] Apply firewall rules for the custom port before binding the daemon.
- [x] Save active firewall chains via `netfilter-persistent save` to survive reboots.
- [x] Disable root password login in favor of SSH public-key authentication where feasible.
