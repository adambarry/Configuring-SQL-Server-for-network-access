# Configuring SQL Server for network access
> Inspiration: https://superuser.com/a/1555144/2119739 and https://blog.codeinside.eu/2019/07/31/sql-server-named-instances-and-the-windows-firewall/

> For reference, access to databases running on Microsoft SQL Server is achieved via **Microsoft SQL Server Management Studio** (SSMS).

On the Windows Server running the SQL Server-instance:
1. Open "SQL Server configuration manager"
2. Expand "SQL Server network configuration", and select "Protocols for {instance name}"
3. Ensure that `TCP/IP` is enabled. If it's not, double-click somewhere in the row to open the "TCP/IP" dialog from where you can modify the value on the "protocol" tab.

> If you're using a **named instance**, you also need to go to the "IP addresses" tab, locate the "IPALL" section (at the bottom) and specify the "TCP port", e.g. `1435`, as all named instances need to be accessible on separate ports.

> In the examples, we assume that the **hostname** (computer name) running the SQL Server-instance(s) is: `SQLHOST`.

Now:
1. Open "Services"
2. Locate the "SQL Server ({instance name})" service
3. Right-click and select "restart" from the context-menu

The changes which were made above are now active.

If you're using a named SQL Server-instance, e.g. `SQLEXPRESS2019`, in "Services":

1. Locate and double-click the `SQL Server browser` service
2. Set `startup type` to `automatic`
3. Click "apply"
4. Click "start"


# Configure Windows Firewall to allow connections to SQL Server
> It appears that SSMS-connections to the SQL Server are considered part of the "public network" firewall rules, if the host isn't enrolled in a domain.

On the Windows Server running the SQL Server-instance, choose the applicable route based on whether you are using a **default instance** or **named instance**, after which you will be able to connect to the SQL Server using SSMS from another host-machine:


## Default instance
If the SQL Server-instance is the "default instance", you need to:

1. Run `cmd`
2. Open PowerShell by executing `powershell` in the command prompt
3. Copy, paste and execute the following commands:

```
New-NetFirewallRule -DisplayName "SQL Server default-instance" -Direction Inbound -LocalPort 1433 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "SQL Server browser service" -Direction Inbound -LocalPort 1434 -Protocol UDP -Action Allow
```

You can now access the SQL Server-instance via SSMS via the "server name": `{hostname}`, e.g. `SQLHOST`.


## Named instance(s)
When running one or more named instances, e.g. `SQLEXPRESS2014`, `SQLEXPRESS2019`, `SQLEXPRESS2022`, there are a couple of additional steps to go through, which will are detailed in the following.

First, you need to create a firewall-rule that enables external communication with the host's "SQL Server browser" service, which is used to query the real TCP port of the named instance. To do this:

1. Run `cmd`
1. Copy, paste and execute the following command:

```
netsh advfirewall firewall add rule name = "SQL Server Browser service" dir = in protocol = udp action = allow localport = 1434 remoteip = localsubnet profile = DOMAIN,PRIVATE,PUBLIC
```

Then you need to run `cmd`, and for **each named instance**, you need to:

1. Modify (the values for `name` and `localport`), copy, paste and execute the following command:

```
netsh advfirewall firewall add rule name = "{instance name}" dir = in protocol = tcp action = allow localport = {port, e.g. 1435} remoteip = localsubnet profile = DOMAIN,PRIVATE,PUBLIC
```

> You can set the `{instance name}` to whatever you like. The instance name is just my preference.

You can now access the SQL Server-instance via SSMS via the "server name": `{hostname}\{instance name}`, e.g. `SQLHOST\SQLEXPRESS2019`.

> You may need to restart the computer in order for the changes to take effect.
