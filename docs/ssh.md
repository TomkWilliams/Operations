# SSH Setup for Operations Workspace

This guide explains how to set up SSH keys and passwordless SSH access between Windows and Linux systems for the user `tomkw`.

## Generating a New SSH Key (on Windows)

1. Open PowerShell and run:
   ```powershell
   ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_local_tomkw -C "local ssh for tomkw"
   ```
   Press Enter to skip a passphrase for passwordless login.

2. Copy both the private and public key files to the client system’s `~/.ssh` directory for `tomkw`.

## Setting Up SSH Access

### On the Linux System
1. Append the public key to `~/.ssh/authorized_keys`:
   ```bash
   cat ~/.ssh/id_ed25519_local_tomkw.pub >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```
2. (Optional) Add to `~/.ssh/config`:
   ```
   Host win-local-tomkw
     HostName <windows-ip-or-hostname>
     User tomkw
     IdentityFile ~/.ssh/id_ed25519_local_tomkw
     IdentitiesOnly yes
   ```


### On the Windows System

**Important for Admin Users:**
If your Windows account (e.g., Tomkw) is in the Administrators group, you must place your public key in:
**Note on Username Capitalization:**
When connecting to your Windows system via SSH, use the exact capitalization of your Windows username as it appears in C:\Users (e.g., `Tomkw`). Using the wrong case (e.g., `tomkw`) may cause authentication to fail or not match the correct profile.

```
C:\ProgramData\ssh\administrators_authorized_keys
```

Steps:
1. Append the public key (from `id_ed25519_local_tomkw.pub`) as a single line to `C:\ProgramData\ssh\administrators_authorized_keys`.
2. Set permissions so only Administrators and SYSTEM have access:
  ```powershell
  icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r /grant "Administrators:F" "SYSTEM:F"
  ```
3. Restart the SSH server:
  ```powershell
  Restart-Service sshd
  ```
4. (Optional) Add to `C:\Users\Tomkw\.ssh\config`:
  ```
  Host linux-local-tomkw
    HostName <linux-ip-or-hostname>
    User tomkw
    IdentityFile ~/.ssh/id_ed25519_local_tomkw
    IdentitiesOnly yes
  ```

#### What Succeeded
- Passwordless SSH from Linux to Windows as an admin user works when the public key is in `C:\ProgramData\ssh\administrators_authorized_keys` and permissions are correct.
- The same key can be used for both directions if present in the correct authorized_keys file on each system.
- Verbose SSH output (`ssh -vvv`) confirmed successful public key authentication.

## Copying Keys with SCP

From Windows to Linux:
```powershell
scp C:\Users\Tomkw\.ssh\id_ed25519_local_tomkw* tomkw@linux-host:~/.ssh/
```

## Testing SSH

- From Windows:
  ```powershell
  ssh linux-local-tomkw
  ```
- From Linux:
  ```bash
  ssh win-local-tomkw
  ```

## Notes
- Use different SSH keys for GitHub and local/server access for better security.
- Ensure the SSH server is running and port 22 is open on both systems.
- You can use either the hostname or IP address for connections.
