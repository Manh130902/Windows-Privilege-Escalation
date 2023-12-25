# 1.1 System Enumeration

```
getuid
sysinfo
```

in windows

```
systeminfo | findstr /B /C: "OS Name" /C: "OS Version" /C:"System Type"

# wmic ( windows manager instrumentation command line )
# qfe ( quick fix engineering )
# xem thông tin các bản vá

wmic qfe get Caption,Description,HotFixID,InstalledOn
wmic logicaldisk get caption,description,providername
```

# 1.2 user enumeration

in Windows:

```
whoami
whoami /priv
whoami /groups
net user
net user <specific user>
net localgroup <group>
```

# 1.3 Network Enumeration

```
ipconfig /all
arp -a
route print
netstat -ano
```

# 1.4 Password Enumeration

```
findstr /si password *.txt *.ini *.config <looks at the current directory>
findstr /spin "password" *.*
%WINDIR%\Panther\Unattend\Unattended.xml
%WINDIR%\Panther\Unattended.xml
dir c:\ /s /b | findstr /si *vnc.ini
```

# 1.5 AV Enumeration

Service controller:

```
sc query windefend
sc queryex type= service

```

Firewall:

```
netsh advfirewall firewall dump
netsh firewall show state
netsh firewall show config
```

# Automated Tools

Executables:

```
winpeas.exe
Seatbelt.exe (compile)
Watson.exe (compile)
SharpUp.exe (compile)

```

PowerShell:

```
Sherlock.ps1
PowerUp.ps1
jaws-enum.ps1

```

Other:

```
windows-exploit-suggester.py (local)
exploit suggester (metasploit)

```
![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/9f27a40d-b57c-4034-8012-141fd20b0f3c)
