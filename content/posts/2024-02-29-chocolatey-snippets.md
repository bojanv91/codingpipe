---
title: "Chocolatey Package Manager Snippets"
date: 2024-02-29
dateUpdated: Last Modified
permalink: /posts/chocolatey-cheatsheet/
tags:
  - Snippets
layout: layouts/post.njk
---

## Installing Chocolatey Windows Package Manager

Start PowerShell as Administrator and run this command:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;

iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.powershell'))

choco feature enable -n=allowGlobalConfirmation

# Make `refreshenv` available right away, by defining the $env:ChocolateyInstall
# variable and importing the Chocolatey profile module.
# Note: Using `. $PROFILE` instead *may* work, but isn't guaranteed to.
# See: https://stackoverflow.com/questions/46758437/how-to-refresh-the-environment-of-a-powershell-session-after-a-chocolatey-instal
$env:ChocolateyInstall = Convert-Path "$((Get-Command choco).Path)\..\.."   
Import-Module "$env:ChocolateyInstall\helpers\chocolateyProfile.psm1"

# refreshenv is now an alias for Update-SessionEnvironment
# (rather than invoking refreshenv.cmd, the *batch file* for use with cmd.exe)
# This should make git.exe accessible via the refreshed $env:PATH, so that it
# can be called by name only.
# See: https://stackoverflow.com/questions/46758437/how-to-refresh-the-environment-of-a-powershell-session-after-a-chocolatey-instal
refreshenv
```

## Installing common Chocolatey packages

```powershell
# PostgreSQL unsupervised with en_US collation:
choco install postgresql14 --params "/Password:postgres /Port:5432" --ia "--enable-components server,commandlinetools --superaccount postgres --locale us"

# RabbitMQ (for troubleshooting, see this: https://bojanveljanovski.com/posts/rabbitmq-cheatsheet/)
choco install rabbitmq

# AWS CLI (for usage, see this: https://bojanveljanovski.com/posts/aws-cli-cheatsheet/)
choco install awscli

# Node Version Manager for Windows (for usage, see this: https://bojanveljanovski.com/posts/npm-nvm-cheatsheet/)
choco install nvm             

# Let's Encrypt windows tool
choco install win-acme

# IIS URL Rewrite extension
choco install urlrewrite      

# .NET Core for Windows Hostinbg - various versions
choco install dotnetcore-windowshosting --version=2.2.2
choco install dotnet-6.0-windowshosting
choco install dotnet-8.0-aspnetruntime
choco install dotnet-8.0-runtime

# .NET Full Framework
choco install netfx-4.8

# Database management tool
choco install dbeaver        

# Notepad++
choco install notepadplusplus
```
