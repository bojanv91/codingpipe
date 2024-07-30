---
title: "RabbitMQ Command Snippets"
date: 2024-02-29
dateUpdated: Last Modified
permalink: /posts/rabbitmq-cheatsheet/
tags:
  - Snippets
  - RabbitMQ
layout: layouts/post.njk
---

## Installing RabbitMQ in Windows using Chocolatey

Start PowerShell as Administrator and run this command, which installs RabbitMQ including the management plugin at port 15672:
```powershell
choco install rabbitmq --version 3.12.10
```

*If you have issues using the RabbitMQ CLI, please refer to the "Troubleshooting common issues" section of this post.*
## Using rabbitmqctl

Enable the management plugin:
```powershell
rabbitmq-plugins enable rabbitmq_management
```

Create user as administrator with permissions to all virtual hosts:
```powershell
# Create new user
rabbitmqctl add_user username password123

# Make user an administrator
rabbitmqctl set_user_tags username administrator

# Grant permissions for all vhosts
rabbitmqctl set_permissions -p / username ".*" ".*" ".*"
```

List users:
```powershell
rabbitmqctl list_users
```

Grants permissions for a user in a virtual host:
```powershell
rabbitmqctl set_permissions -p "custom-vhost" "username" ".*" ".*" ".*"
```

Revoke permissions of a user in a virtual host:
```powershell
rabbitmqctl clear_permissions -p "custom-vhost" "username"
```

List all virtual hosts:
```powershell
rabbitmqctl list_vhosts
```

Create a vhost:
```powershell
rabbitmqctl add_vhost "custom-vhost"
```

Delete a vhost:
```powershell
rabbitmqctl delete_vhost "custom-vhost"
```

## Troubleshooting common issues with RabbitMQ CLI

If RabbitMQ CLI tools are not accessible in your command prompt, add it to PATH and refresh the session:
```powershell
# First, check if rabbitmq CLI tools are accessible
rabbitmqctl version

# If rabbitmqctl is not found, then add it to PATH and refresh the session
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

# Now this should work
rabbitmqctl version
```

Fix the RabbitMQ Erland cookie mismatch (https://www.rabbitmq.com/cli.html#erlang-cookie):
```powershell
Copy-Item -Path C:\Windows\System32\config\systemprofile\.erlang.cookie -Destination C:\Users\$Env:UserName -force
```

Troubleshooting commands:
```powershell
rabbitmqctl report
rabbitmqctl status
```

Test RabbitMQ aliveness with the default local user 'guest/guest':
```powershell
iwr -Uri 'http://localhost:15672/api/aliveness-test/%2F' -Headers @{ Authorization = "Basic "+ [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("guest:guest")) } -UseBasicParsing
```

## Additional Resources

- https://www.rabbitmq.com/docs/cli
