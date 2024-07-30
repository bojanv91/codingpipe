---
title: "PowerShell Command Snippets"
date: 2023-12-05
dateUpdated: Last Modified
permalink: /posts/powershell-snippets/
tags:
  - PowerShell
  - Snippets
layout: layouts/post.njk
---

Add a Windows user and grant remote desktop access:
```powershell
$Password = Read-Host -AsSecureString   
#<enter your new password>
New-LocalUser "MyRDPUser" -Password $Password

Add-LocalGroupMember -Group "Remote Desktop Users" -Member "MyRDPUser"
```

Change RDP user password, and set account to never expire:
```powershell
$Password = Read-Host -AsSecureString
#<enter your new password>

$UserAccount = Get-LocalUser -Name "MyRDPUser"
$UserAccount | Set-LocalUser -Password $Password -AccountNeverExpires -PasswordNeverExpires 1
```

Grant admin rights to a Windows user:
```powershell
Add-LocalGroupMember -Group "Administrators" -Member "MyNewUser"
```

Add firewall rules - allow inbound ports:
```powershell
New-NetFirewallRule -DisplayName "Port 80" -Direction Inbound -LocalPort 80 -Protocol TCP -Action Allow

New-NetFirewallRule -DisplayName "Port 443" -Direction Inbound -LocalPort 443 -Protocol TCP -Action Allow
```

Install IIS and dependencies:
```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
Install-WindowsFeature web-scripting-tools
```

Restart an IIS website:
```powershell
Stop-IISSite -Name "YourWebSite"
Start-IISSite -Name "YourWebSite"
```

Create/manage IIS website binding with AppPool:
```powershell
# Create new IIS site
New-IISSite -Name "mywebsite" -BindingInformation "*:80:localhost" -PhysicalPath "C:\inetpub\wwwroot\mywebsite"

# Add HTTPS binding
New-IISSiteBinding -Name "mywebsite" -BindingInformation "*:443:localhost" -CertificateThumbPrint "<YOUR_CERT_THUMBPRINT>" -CertStoreLocation "Cert:\LocalMachine\My" -Protocol https -SslFlag 1

# Create AppPool
New-Item -Path "IIS:\AppPools" -Name "mywebsite" -Type AppPool

# "v2.0", "v4.0" and "" (for no managed code)
Set-ItemProperty -Path "IIS:\AppPools\mywebsite" -name "managedRuntimeVersion" -value ""
Set-ItemProperty -Path "IIS:\AppPools\mywebsite" -name "autoStart" -value $true
Set-ItemProperty -Path "IIS:\AppPools\mywebsite" -name "processModel" -value @{identitytype="ApplicationPoolIdentity"}

# Assign the application pool to a website
Set-ItemProperty -Path "IIS:\Sites\mywebsite" -name "applicationPool" -value "mywebsite"
```

Remove app pool:
```powershell
Remove-Item -Path "IIS:\AppPools\mywebsite" -Recurse -Force -ErrorAction SilentlyContinue
```

Grant access to the "/logs" folder for an IIS AppPool:
- This script assumes that the AppPool name has the same name as the Website name. In order for a given ASP.NET website to be able to write logs to the "/logs" folder, the AppPool needs to have access to that folder. 
```powershell
function GrantWebsiteFolderPermissions { 
    param ($websiteName) 
                  
    $Path = "C:/inetpub/wwwroot/$websiteName/logs"
    $Acl = Get-Acl $Path
    $Ar = New-Object System.Security.AccessControl.FileSystemAccessRule("IIS AppPool\$websiteName", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
    $Acl.SetAccessRule($Ar)
    Set-Acl $Path $Acl
}

GrantWebsiteFolderPermissions -websiteName "mywebsite"
```

Install Chocolatey Windows Package Manager:
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;

iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.powershell'))

choco feature enable -n=allowGlobalConfirmation
```

Create scheduled tasks using Windows Task Scheduler:
```powershell
$Action = New-ScheduledTaskAction -Execute ".\runBackupProcedure.bat" -WorkingDirectory "C:\YourStartInPath"
$Triggers = @(      # Add two daily triggers at different times
    $(New-ScheduledTaskTrigger -Daily -At 3PM),
    $(New-ScheduledTaskTrigger -Daily -At 6AM)
)
$Settings = New-ScheduledTaskSettingsSet -HistoryEnabled $true
$Principal = New-ScheduledTaskPrincipal -UserId "NT AUTHORITY\SYSTEM" -LogonType ServiceAccount

# Register the task
Register-ScheduledTask -Action $Action -Trigger $Triggers -TaskName "BackupDatabaseToS3" -Description "Runs BackupDatabaseToS3 daily" -Settings $Settings -Principal $Principal

# Get task info
Get-ScheduledTask -TaskName "BackupDatabaseToS3" | Get-ScheduledTaskInfo
```

```batch
:: runBackupProcedure.bat
powershell.exe -ExecutionPolicy Bypass -File ".\backupDatabase.ps1"
powershell.exe -ExecutionPolicy Bypass -File ".\uploadBackupToS3.ps1"
```
