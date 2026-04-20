# Setting Up SSHD on Linux

This guide covers installing and enabling the OpenSSH server (sshd) on a Linux system, as well as configuring the firewall to allow inbound SSH connections.

## 1. Install OpenSSH Server

```bash
sudo apt update
sudo apt install -y openssh-server
```

## 2. Enable and Start the SSH Service

```bash
sudo systemctl enable --now ssh
```
This ensures the SSH service starts automatically on boot and is running now.

## 3. Allow SSH Through the Firewall (UFW)

If you use UFW (Uncomplicated Firewall), allow SSH traffic:

```bash
sudo ufw allow ssh
```

## 4. Check SSH Service Status

```bash
sudo systemctl status ssh
```

## 5. Find Your Machine's IP Address

```bash
ip address
```
Look for the `inet` entry under your network interface (e.g., `wlo1`).

## 6. Connect from Another Machine

```bash
ssh yourusername@your_machine_ip
```

---

**Security Tips:**
- Use SSH keys for authentication instead of passwords.
- Disable root login via SSH by editing `/etc/ssh/sshd_config`.
- Change the default SSH port for added security (optional).
- Regularly update your system and monitor SSH logs.
