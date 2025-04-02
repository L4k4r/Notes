-----
There may be scenarios where HTTP, HTTPS or SMB are unavailable, if this is the case, we can use PowerShell Remoting, aka WinRM to perform file transfer options.

PowerShell Remoting allows us to execute scripts or commands on a remote computer using PowerShell sessions. By default, enabling PowerShell remoting creates both HTTP and an HTTPS listener. The listeners run on default ports `TCP/5985` for **HTTP** and `TCP/5986` for **HTTPS**.

To create a PowerShell Remoting session on a computer, we will need one of these:
- Administrative access
- Be a member of the Remote Management Users group
- Have explicit permissions for PowerShell Remoting in the session configuration.

**Example**

```powershell
Test-NetConnection -ComputerName DATABASE01 -Port 5985
```
```powershell
ComputerName     : DATABASE01
RemoteAddress    : 192.168.1.101
RemotePort       : 5985
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.1.100
TcpTestSucceeded : True
```

Because this session already has privileges over `DATABASE01`, we don't need to specify credentials. In the example below, a session is created to the remote computer named `DATABASE01` and stores the results in the variable named `$Session`.

```powershell
$Session = NewPSSession -ComputerName DATABASE01
```

We can use the `Copy-Item` cmdlet to copy a file from our local machine `DC01` to the `DATABASE01` session we have `$Session` or vice versa.

 **Copy samplefile.txt from our Localhost to the DATABASE01 Session**
```powershell
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```

**Copy DATABASE.txt from DATABASE01 Session to our Localhost**
```powershell
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```