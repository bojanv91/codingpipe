---
title: "Developer Command Snippets"
pubDatetime: 2024-02-29
modDatetime: 2026-04-05
description: "A consolidated collection of command snippets I use regularly — PowerShell, PostgreSQL, RabbitMQ, AWS CLI, NPM/NVM, Chocolatey, and EF Core."
slug: developer-command-snippets
tags:
  - reference
draft: false
---

## Table of Contents

## PowerShell

### Windows User Management

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

### Firewall Rules

Add firewall rules — allow inbound ports:

```powershell
New-NetFirewallRule -DisplayName "Port 80" -Direction Inbound -LocalPort 80 -Protocol TCP -Action Allow

New-NetFirewallRule -DisplayName "Port 443" -Direction Inbound -LocalPort 443 -Protocol TCP -Action Allow
```

### IIS Management

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

Grant access to the "/logs" folder for an IIS AppPool (assumes the AppPool name matches the Website name):

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

### Scheduled Tasks

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

---

## PostgreSQL

### Environment Variables

Set the PostgreSQL environment variables in your PowerShell session:

```powershell
$env:PGHOST = "localhost"
$env:PGPORT = 5432
$env:PGUSER = "postgres"
$env:PGPASSWORD = "postgres"
```

### Backup and Restore

```powershell
# Backup
pg_dump --format custom --no-owner --no-privileges --no-acl --host localhost --port 5432 --username "postgres" --dbname mydb --file "C:/backups/mydb.backup"

# Restore - Step 1: Create the database if it doesn't exist
psql --host localhost --port 5432 --username "postgres" --dbname postgres -c "CREATE DATABASE mydb WITH OWNER postgres ENCODING 'UTF8';"

# Restore - Step 2: Drop the public schema in the target database
psql --host localhost --port 5432 --username "postgres" --dbname mydb -c "DROP SCHEMA IF EXISTS public CASCADE;"

# Restore - Step 3: Restore the backup file
pg_restore --format custom --no-owner --no-privileges --no-acl --clean --if-exists --exit-on-error --single-transaction --host localhost --port 5432 --username "postgres" --dbname mydb "D:/backups/mydb-2024-09-01-1150.backup"
```

Restore backup only in a specific schema:

```powershell
pg_restore --schema "schema_name_1" --format custom --no-owner --no-privileges --single-transaction --dbname mydb "C:/backups/mydb.backup"
```

### Database and User Management

```powershell
# List all databases
psql --command \l

# Create database
psql --command "CREATE DATABASE mydb;"
psql --command "CREATE DATABASE mydb OWNER testuser ENCODING 'UTF8';"

# Create user
psql --command "CREATE ROLE testuser NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT LOGIN NOREPLICATION NOBYPASSRLS PASSWORD 'testpsw';"

# List users
psql --command \du

# Create schema with authorization
psql --command "CREATE SCHEMA IF NOT EXISTS schema3 AUTHORIZATION testuser2;" mydb

# List all schemas in database
psql --command \dn mydb
```

### Iterate Over All Databases

```powershell
$databases = psql --tuples-only --no-align --command "SELECT datname FROM pg_database WHERE datistemplate = false"

foreach ($db in $databases)
{
    if ($db) {
        Write-Host "Processing database: $db"
    }
}
```

### Additional PostgreSQL References

- <https://wiki.postgresql.org/wiki/Operations_cheat_sheet>
- <https://www.commandprompt.com/education/postgresql-basic-psql-commands/>
- <https://www.postgresqltutorial.com/postgresql-cheat-sheet/>
- <https://severalnines.com/blog/performance-cheat-sheet-postgresql>

---

## RabbitMQ

### Installation

Install RabbitMQ in Windows using Chocolatey (includes the management plugin at port 15672):

```powershell
choco install rabbitmq --version 3.12.10
```

### Using rabbitmqctl

Enable the management plugin:

```powershell
rabbitmq-plugins enable rabbitmq_management
```

Create user as administrator with permissions to all virtual hosts:

```powershell
rabbitmqctl add_user username password123
rabbitmqctl set_user_tags username administrator
rabbitmqctl set_permissions -p / username ".*" ".*" ".*"
```

List users:

```powershell
rabbitmqctl list_users
```

Virtual host management:

```powershell
# List all virtual hosts
rabbitmqctl list_vhosts

# Create a vhost
rabbitmqctl add_vhost "custom-vhost"

# Delete a vhost
rabbitmqctl delete_vhost "custom-vhost"

# Grant permissions for a user in a virtual host
rabbitmqctl set_permissions -p "custom-vhost" "username" ".*" ".*" ".*"

# Revoke permissions
rabbitmqctl clear_permissions -p "custom-vhost" "username"
```

### Troubleshooting RabbitMQ CLI

If RabbitMQ CLI tools are not accessible, add to PATH and refresh the session:

```powershell
Function Set-PathVariable {
    param(
        [string]$AddPath
    )
    if (Test-Path $AddPath){
        $regexAddPath = [regex]::Escape($AddPath)
        $arrPath = $env:Path -split ';' | Where-Object {$_ -notMatch "^$regexAddPath\\?"}
        $envPathToSet = ($arrPath + $AddPath) -join ';'
        [Environment]::SetEnvironmentVariable("Path", $envPathToSet, "Machine")
    } else {
        Throw "'$AddPath' is not a valid path."
    }
}

Set-PathVariable -AddPath "C:\Program Files\RabbitMQ Server\rabbitmq_server-3.12.10\sbin"
refreshenv
rabbitmqctl version
```

Fix the Erlang cookie mismatch:

```powershell
Copy-Item -Path C:\Windows\System32\config\systemprofile\.erlang.cookie -Destination C:\Users\$Env:UserName -force
```

Troubleshooting and aliveness test:

```powershell
rabbitmqctl report
rabbitmqctl status

# Test aliveness with default local user 'guest/guest'
iwr -Uri 'http://localhost:15672/api/aliveness-test/%2F' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("guest:guest")) } -UseBasicParsing
```

---

## AWS CLI

### Installation

```powershell
choco install awscli
```

### Profiles

Always use profiles — leave the default profile unconfigured to prevent mistakes:

```powershell
aws configure --profile profilename
```

### S3 with Backblaze B2

Configure the `backblaze` profile:

```powershell
$AWS_ACCESS_KEY_ID = "<BackblazeKeyID>"
$AWS_SECRET_ACCESS_KEY = "<BackblazeAppKey>"

aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID --profile backblaze; `
    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY --profile backblaze
```

Test and upload:

```powershell
# Test the integration
aws s3 ls --endpoint-url=$BackblazeS3EndpointUrl --profile backblaze

# Upload to S3-compatible bucket
aws s3 cp "C:/path/to/backup/" s3://custom-bucket-name-in-backblaze/ --recursive --profile backblaze --endpoint-url=$BackblazeS3EndpointUrl
```

---

## NPM and NVM

### Installation via Chocolatey

```powershell
choco install nvm
nvm install latest  # or replace 'latest' with a specific version
```

### Using NVM

```powershell
nvm current              # Display currently active version
nvm install <version>    # Install specific version
nvm list                 # List available installations
nvm use <version>        # Switch to a version
```

Switching versions:

```powershell
nvm use 21.6.2
npm install     # uses the 21.6.2 version
nvm use 18.16.0
npm install     # uses the 18.16.0 version
```

### Using NPM

```powershell
npm install     # Install everything in package.json
npm list        # List installed dependency versions
npm outdated    # List only outdated dependencies
```

### Additional NPM/NVM References

- <https://github.com/coreybutler/nvm-windows>
- <https://community.chocolatey.org/packages/nvm>
- <https://github.com/nvm-sh/nvm>

---

## Chocolatey

### Installation

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;

iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.powershell'))

choco feature enable -n=allowGlobalConfirmation

# Make `refreshenv` available right away
$env:ChocolateyInstall = Convert-Path "$((Get-Command choco).Path)\..\.."   
Import-Module "$env:ChocolateyInstall\helpers\chocolateyProfile.psm1"
refreshenv
```

### Common Packages

```powershell
# PostgreSQL
choco install postgresql14 --params "/Password:postgres /Port:5432" --ia "--enable-components server,commandlinetools --superaccount postgres --locale us"

# RabbitMQ
choco install rabbitmq

# AWS CLI
choco install awscli

# Node Version Manager for Windows
choco install nvm             

# Let's Encrypt windows tool
choco install win-acme

# IIS URL Rewrite extension
choco install urlrewrite      

# .NET hosting and runtimes
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

---

## EF Core Migrations

Commands I find myself looking up constantly. Run these from the project directory containing your DbContext. Add `--project <ProjectName>` if running from solution root. Use `--output-dir Data/Migrations` to organize migrations in a specific folder.

```powershell
# Install EF Core CLI globally
dotnet tool install --global dotnet-ef

# Check migration status and list all migrations
dotnet ef migrations list

# Apply pending migrations to database
dotnet ef database update

# Generate new migration with descriptive name
dotnet ef migrations add <MigrationName>
dotnet ef migrations add <MigrationName> --output-dir Data/Migrations

# Drop database
dotnet ef database drop

# Remove all migrations
dotnet ef migrations remove --force

# View DbContext information and connection details
dotnet ef dbcontext info
```
