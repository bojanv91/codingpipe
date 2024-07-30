---
title: "SSH Command Snippets (Windows)"
date: 2024-03-31
dateUpdated: Last Modified
permalink: /posts/ssh-windows-snippets/
tags:
  - Snippets
layout: layouts/post.njk
---

Windows 10 includes an OpenSSH client b default. You can use it directly from the PowerShell or command prompt terminal.

## Connecting to a Linux server

Open a terminal and run the following command:
```
ssh username@12.34.56.78
```

Replace `username` with your Linux server username and `12.34.56.78` with the server’s IP address.
The first time you connect, you’ll be prompted to accept the host key. After that, enter your password to log in.
To log out, run the `exit` command or press `Ctrl+D`.

If your Linux server has a different SSH port, let's say 10322, use the `-p` flag:
```
ssh username@12.34.56.78 -p 10322
```

## Downloading/uploading files between your Windows client and a Linux server

Use [WinSCP](https://winscp.net/eng/download.php) for that.

## Tools

- [PuTTY](https://www.putty.org/)
- [WinSCP](https://winscp.net/eng/download.php)
- [PuTTYgen](https://www.puttygen.com/)

## Additional Resources

- https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui
- https://docs.bitnami.com/aws/faq/get-started/access-ssh-tunnel/
- https://docs.vultr.com/port-forwarding-and-proxying-using-openssh
