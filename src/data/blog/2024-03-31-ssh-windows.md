---
title: "SSH tips on Windows"
pubDatetime: 2024-03-31
description: "Essential SSH configuration tips and troubleshooting guide for Windows users, including key management, connection setup, and common issues."
slug: ssh-windows-snippets
tags:
  - windows
  - security
  - cli
draft: false
---

Windows 11 includes an OpenSSH client b default. You can use it directly from the PowerShell or command prompt terminal.

## Connecting to a Linux server

Open a terminal and run the following command:

```plaintext
ssh username@12.34.56.78
```

Replace `username` with your Linux server username and `12.34.56.78` with the server’s IP address.
The first time you connect, you’ll be prompted to accept the host key. After that, enter your password to log in.
To log out, run the `exit` command or press `Ctrl+D`.

If your Linux server has a different SSH port, let's say 10322, use the `-p` flag:

```plaintext
ssh username@12.34.56.78 -p 10322
```

## Downloading/uploading files between your Windows client and a Linux server

Use [WinSCP](https://winscp.net/eng/download.php) for that.

## Tools

- [PuTTY](https://www.putty.org/)
- [WinSCP](https://winscp.net/eng/download.php)
- [PuTTYgen](https://www.puttygen.com/)

## Additional Resources

- <https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui>
- <https://docs.bitnami.com/aws/faq/get-started/access-ssh-tunnel/>
- <https://docs.vultr.com/port-forwarding-and-proxying-using-openssh>

## Managing SSH Keys with OpenSSH and Bitwarden

### 1. Set up your environment

Create the SSH directory if it doesn't exist:

```powershell
mkdir -p $env:USERPROFILE\.ssh
```

### 2. Generate SSH Key Pair

Open PowerShell and generate an Ed25519 key (recommended):

```powershell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Or RSA if required:

```powershell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

When prompted:

- Save to default location (`%USERPROFILE%\.ssh\id_ed25519` or `id_rsa`)
- Set a strong passphrase

This creates two files:

- Private key: `id_ed25519` (no extension)
- Public key: `id_ed25519.pub`

### 3. Store in Bitwarden

Step 1. Open Bitwarden and create a new Secure Note

Step 2. Title: "SSH Key - [Purpose]" (e.g., "SSH Key - Work Laptop 2024")

Step 3. Add attachments:

- Browse to `%USERPROFILE%\.ssh\`
- Attach both `id_ed25519` and `id_ed25519.pub`

Step 4. Add to notes:

```plaintext
Server: example.com
Username: myuser
Key Type: Ed25519
Created: [Date]
Passphrase: [your-passphrase]

Public Key:
[paste contents of id_ed25519.pub here]

Usage:
1. Download private key
2. Save to ~/.ssh/id_ed25519
3. Set permissions: chmod 600 ~/.ssh/id_ed25519
```

Step 5. Add tags: "ssh", "credentials", "work"
Step 6. Save

### 4. Deploy to Server

Copy your public key to the server:

```powershell
# Method 1: Using ssh-copy-id (if available)
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@example.com

# Method 2: Manual copy
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh username@example.com "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Verify the connection:

```powershell
ssh -i $env:USERPROFILE\.ssh\id_ed25519 username@example.com
```

### 5. Recovery Process

If you need to recover access from a new device:

Step 1. Install OpenSSH client if not present:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

Step 2. Create SSH directory:

```powershell
mkdir -p $env:USERPROFILE\.ssh
```

Step 3. Download private key from Bitwarden
Step 4. Save as `%USERPROFILE%\.ssh\id_ed25519`
Step 5. Set correct permissions:

```powershell
icacls "%USERPROFILE%\.ssh\id_ed25519" /inheritance:r /grant:r "%USERNAME%":"F"
```

Step 6. Test connection:

```powershell
ssh -i $env:USERPROFILE\.ssh\id_ed25519 username@example.com
```

Step 7. After recovery, generate new keys and update servers

### Connecting to Servers with SSH Keys

Step 1. Basic connection with default key location:

```powershell
ssh username@192.168.1.100
```

Step 2. Specify a different port (e.g., 10322):

```powershell
ssh -p 10322 username@192.168.1.100
```

Step 3. Explicitly specify which key to use:

```powershell
ssh -i $env:USERPROFILE\.ssh\id_ed25519 username@192.168.1.100
```

Step 4. For multiple servers/keys, create a config file at `%USERPROFILE%\.ssh\config`:

```plaintext
Host myserver1
    HostName 192.168.1.100
    User username
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host myserver2
    HostName 192.168.1.200
    User otheruser
    Port 10322
    IdentityFile ~/.ssh/id_rsa
```

Then connect using the alias:

```powershell
ssh myserver1
```

> 💡 Tip: After adding your key to a server, you won't need to enter a password. You'll only need the key's passphrase (if you set one).
