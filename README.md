# SSH Key Setup on a Linux Server

## Project Url
https://roadmap.sh/projects/ssh-remote-server-setup

**Role Perspective:** DevSecOps Engineer
**Audience:** Developers, SysAdmins, DevOps/DevSecOps Engineers
**Goal:** Securely configure SSH key-based authentication for Linux servers, replacing or strengthening password-based access.

---

## 1. Why Use SSH Keys?

SSH key-based authentication is more secure than passwords because:

* Resistant to brute-force attacks
* No passwords transmitted over the network
* Supports automation (CI/CD, configuration management)
* Enables fine-grained access control and auditing

**Security Best Practice:** Disable password authentication once SSH keys are verified.

---

## 2. Prerequisites

* A Linux server with SSH access
* A local machine (Linux/macOS/Windows with WSL)
* `ssh` and `ssh-keygen` installed
* Non-root user with sudo privileges (recommended)

---

## 3. Generate an SSH Key Pair (Client Side)

Run the following on your **local machine**:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### Key Type Recommendations

| Type       | Security | Notes                     |
| ---------- | -------- | ------------------------- |
| ed25519    | ⭐⭐⭐⭐⭐    | Modern, fast, recommended |
| rsa (4096) | ⭐⭐⭐⭐     | Legacy compatibility      |

### Prompts Explained

* **File location**: Press Enter to accept default (`~/.ssh/id_ed25519`)
* **Passphrase**: Strongly recommended for human access

This generates:

* **Private key** → `~/.ssh/id_ed25519` (KEEP SECRET)
* **Public key** → `~/.ssh/id_ed25519.pub`

---

## 4. Copy the Public Key to the Server

### Option A: Using `ssh-copy-id` (Recommended)

```bash
ssh-copy-id user@server_ip
```

This automatically:

* Creates `~/.ssh` if missing
* Appends key to `authorized_keys`
* Sets correct permissions

### Option B: Manual Method

1. Display public key:

   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

2. SSH into server:

   ```bash
   ssh user@server_ip
   ```

3. Configure SSH directory:

   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   nano ~/.ssh/authorized_keys
   ```

4. Paste the public key, then:

   ```bash
   chmod 600 ~/.ssh/authorized_keys
   ```

---

## 5. Verify SSH Key Login

From your local machine:

```bash
ssh user@server_ip
```

Expected result:

* Login without password (or only key passphrase)
* No errors or permission warnings

---

## 6. Secure SSH Daemon Configuration (Server Side)

Edit SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

### Recommended Secure Settings

```ini
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
AllowUsers user
```

### Apply Changes

```bash
sudo sshd -t   # Validate config
sudo systemctl restart sshd
```

⚠️ **Important:** Keep one active session open while testing new SSH settings.

---

## 7. Permissions Checklist (Critical for Security)

| Path                   | Permission |
| ---------------------- | ---------- |
| ~/.ssh                 | 700        |
| ~/.ssh/authorized_keys | 600        |
| Private key (local)    | 600        |

Incorrect permissions will cause SSH to ignore keys.

---

## 8. Multiple Keys & Team Access

For teams and automation:

* One key per user/service
* Never share private keys
* Use comments to identify keys

Example:

```text
ssh-ed25519 AAAAC3Nza... deploy@github-actions
```

---

## 9. SSH Agent (Optional but Recommended)

Avoid re-entering passphrases:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---

## 10. DevSecOps Best Practices

* Rotate keys periodically
* Remove unused keys immediately
* Use bastion hosts / jump servers
* Log SSH access (`auditd`, `fail2ban`)
* Enforce MFA where possible
* Store automation keys in secret managers

---

## 11. Common Troubleshooting

### Permission Denied (publickey)

* Check file permissions
* Verify correct user
* Confirm key exists in `authorized_keys`

### SSH Ignoring Keys

```bash
ssh -v user@server_ip
```

Look for:

* `Offering public key`
* `Authentication succeeded`

---

## 12. Conclusion

SSH key authentication is a foundational security control in DevSecOps. Proper key management, hardened SSH configuration, and periodic audits significantly reduce attack surface while enabling secure automation.

---

**Document Owner:** DevSecOps Team
**Last Updated:** 2025
