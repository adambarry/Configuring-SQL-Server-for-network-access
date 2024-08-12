# Configuring SQL Server for network access
> Inspiration: https://superuser.com/a/1555144/2119739

On the Windows Server running the SQL Server-instance:
1. Open "SQL Server configuration manager"
2. Expand "SQL Server network configuration", and select "Protocols for {instance name}"
3. Ensure that `TCP/IP` is enabled

Now:
1. Open "Services"
2. Locate the "SQL Server ({instance name})" service
3. Right-click and select "restart" from the context-menu

The changes which were made above are now active.

If you're using a named SQL Server-instance, e.g. "SQLEXPRESS", in "Services":
1. Locate and double-click the "SQL Server browser" service
2. Set `startup type` to `automatic`
3. Click "apply"
4. Click "start"

If no firewall is active (or causing problems), you are now able to access the SQL Server-instance from another host-machine using "SQL Server Management Studio" (SSMS).


# Configure Windows Firewall to allow connections to SQL Server
> It appears that SSMS-connections to the SQL Server are considered part of the "public network" firewall rules.

On the Windows Server running the SQL Server-instance, choose the applicable route based on whether you are using a default or named SQL Server-instance, after which you will be able to connect to the SQL Server using SSMS from another host-machine:

## Default-instance
1. Run `CMD`
2. Open PowerShell by executing `powershell` in the command prompt
3. Paste and execute the following commands:

```
New-NetFirewallRule -DisplayName "SQL Server default-instance" -Direction Inbound -LocalPort 1433 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "SQL Server Browser service" -Direction Inbound -LocalPort 1434 -Protocol UDP -Action Allow
```

## Named-instance
1. Open "Windows Defender Firewall with advanced security"
2. Select "inbound rules"
3. Click "new rule":
    1. Rule type: `Program`
    2. Program: This program path, e.g.: `%ProgramFiles%\Microsoft SQL Server\{SQL Server version}\MSSQL\Binn\sqlservr.exe` (navigate to the correct folder according to the SQL Server version)
    3. Action: `Allow the connection`
    4. Profile: Select all options, i.e. `domain`, `private`, `public`
    5. Name: `SQL Server`
4. Click "finish"
5. Click "new rule":
    1. Rule type: `Program`
    2. Program: This program path: `C:\Program Files (x86)\Microsoft SQL Server\90\Shared\sqlbrowser.exe` (the default destination for the SQL Server browser)
    3. Action: `Allow the connection`
    4. Profile: Select all options, i.e. `domain`, `private`, `public`
    5. Name: `SQL Server browser`
6. Click "finish"
