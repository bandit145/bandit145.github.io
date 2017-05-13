---
layout: post
title: "Stop Using SMB1"
date: 2017-05-12 16:30
categories: security windows smb powershell
---
The NHS (National Health Service) in the U.K. has suffered a major ransomeware attack affecting a large number of hospitals in the U.K. The attacker used <a href=https://technet.microsoft.com/en-us/library/security/ms17-010.aspx>this</a> (patched) exploit.

This got me wondering about SMB1 and I was checked my homelab Windows Server 2012R2 file server and was surprised to see SMB1 enabled, it turns out SMB1 is still enabled by default.... There are cases where you have some legacy thing that only supports SMB1 but that's the exception. So, How do you disable SMB1 in Windows Server 2012+?

1. Check your SMB Server Configuration:
	```
	&>Get-SmbServerConfiguration | Select EnableSMB1Protocol

	EnableSMB1Protocol
	------------------
            	False
	```

2. Disable:
	```
	 Set-SmbServerConfiguration -EnableSMB1Protocol $false -Confirm:$false
	```

Script:

This will disable SMB1 on the servers fed to it.
```
param(
[parameter(Mandatory=$true)]
[String[]]$Computers
)

foreach($computer in $Computers){
    $session = New-PSSession -ComputerName $computer
    Invoke-Command -Session $session -ScriptBlock{
        if((Get-SmbServerConfiguration).EnableSMB1Protocol){
            Set-SmbServerConfiguration -EnableSMB1Protocol $false -Confirm:$false
            Write-Host "SMB1 disabled on $env:computername"
        }
    }
}

```
Run it like this:

```
./script.ps1 -Computers computer1,computer2,etc.
```


Bam! that's it. It would probably be better to remove it entirely though.

We can do that easily! 

1.	Check to see if the FS-SMB1 feature is installed:

	```
	&>Get-WindowsFeature -Name FS-SMB1

	Display Name                                            Name                       Install State
	------------                                            ----                       -------------
	[X] SMB 1.0/CIFS File Sharing Support                   FS-SMB1                        Installed

	```

2. Remove it:

	```
	&>Remove-WindowsFeature -Name FS-SMB1
	```
	Note: The server will require a restart to fully complete the operation.

Script:

```
param(
[parameter(Mandatory=$true)]
[String[]]$Computers
)

foreach($computer in $Computers){
    $session = New-PSSession -ComputerName $computer
    Invoke-Command -Session $session -ScriptBlock{
        if((Get-WindowsFeature -Name "FS-SMB1").InstallState -eq "Installed"){
            Remove-WindowsFeature -Name "FS-SMB1" | Out-Null
            Write-Host "FS-SMB1  removed on $env:computername"
        }
    }
}
```
Run it like this:

```
./script.ps1 -Computers computer1,computer2,etc.
```

If you are using ansible you might want to make sure the role is removed when you run your initial playbook against it. You can do so like this:

```
- name: Remove FS-SMB1 feature
  win_feature: name: "FS-SMB1" state: absent restart: yes
```