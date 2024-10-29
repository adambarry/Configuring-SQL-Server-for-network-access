# Configuring SQL Server for network access
> Inspiration: https://superuser.com/a/1555144/2119739 and https://blog.codeinside.eu/2019/07/31/sql-server-named-instances-and-the-windows-firewall/

On the Windows Server running the SQL Server-instance:
1. Open "SQL Server configuration manager"
2. Expand "SQL Server network configuration", and select "Protocols for {instance name}"
3. Ensure that `TCP/IP` is enabled. If it's not, double-click somewhere in the row to open the "TCP/IP" dialog from where you can modify the value on the "protocol" tab.

> If you're using a named instance, you also need to go to the "IP addresses" tab, locate the "IPALL" section (at the bottom) and specify the "TCP port", e.g. `1435`, as all named instances need to be accessible on separate ports.

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
3. Copy, paste and execute the following commands:

```
New-NetFirewallRule -DisplayName "SQL Server default-instance" -Direction Inbound -LocalPort 1433 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "SQL Server Browser service" -Direction Inbound -LocalPort 1434 -Protocol UDP -Action Allow
```

## Named-instance(s)
When running one or more named instances, there are a couple of other steps to go through.

First, you need to create a firewall-rule to open UDP port `1434` which is used to query the real TCP port of the named instance. To do this:

1. Run `cmd`
1. Copy, paste and execute the following command:

```
netsh advfirewall firewall add rule name = "SQL Server UDP port" dir = in protocol = udp action = allow localport = 1434 remoteip = localsubnet profile = DOMAIN,PRIVATE,PUBLIC
```

Next, you need to enable a passage for the "SQL browser" through the firewall. To do this:

1. Open "Windows Defender Firewall with advanced security"
1. Select "inbound rules"
1. Click "new rule":
    1. Rule type: `Program`
    1. Program: This program path: `C:\Program Files (x86)\Microsoft SQL Server\90\Shared\sqlbrowser.exe` (the default destination for the SQL Server browser)
    1. Action: `Allow the connection`
    1. Profile: Select all options, i.e. `domain`, `private`, `public`
    1. Name: `SQL Server browser`
1. Click "finish"

> You probably need to restart the computer in order for the changes to take effect.

Then for each named instance, you need to:

1. Run `cmd`
1. Modify (`name` and `localport`), copy, paste and execute the following command:

```
netsh advfirewall firewall add rule name = "SQL Server {version}" dir = in protocol = tcp action = allow localport = {port, e.g. 1435} remoteip = localsubnet profile = DOMAIN,PRIVATE,PUBLIC
```
