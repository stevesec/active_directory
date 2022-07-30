# Best Practices for Samba Active Directory (Same as Windows Active Directory)

- Command for moving SSH Keys to Linux from Windows:
```
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh <username>@{IP Address of DC} "cat >> .ssh/authorized_keys"
```
- Install and Configure Unattended Upgrades for Linux System Administration
```
sudo apt install unattended-upgrades apt-listchanges
```
- Turn on unattended security updates:
```
sudo dpkg-reconfigure -plow unattended-upgrades
```
- Install Microsoft Remote Server Administration Tools (RSAT) (Command Line)
```
dism /online /add-capability /CapabilityName:Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 /CapabilityName:Rsat.Dns.Tools~~~~0.0.1.0 /CapabilityName:Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
```
- [Best Practices for Securing Active Directory](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/best-practices-for-securing-active-directory)

- [Installing LAPS for Samba Active Directory](https://samba.tranquil.it/doc/en/samba_advanced_methods/samba_configure_laps.html)
