# Deployment of Samba Active Directory

- Install Ubuntu Server on Proxmox PVE or Virtual Host

## Set hostname 

```
hostnamectl set-hostname dc1
```

- Open /etc/hosts with preferred text editor
 - Append this to the end of the hosts file
```
#change FQDN to whatever you would like 
<IP Address of Host> dc1.example.lan dc1
```
- Verify FQDN
```
#verify FQDN
hostname -f

#verify FQDN is resolved to the Samba IP address
ping -c3 dc1.example.lan
```
## Restart disable the DNS Resolver
- Ubuntu DNS Configuration is not suitable for Samba Active Directory.
```
systemctl disable --now systemd-resolved

unlink /etc/resolv.conf
```
- Then, create a new resolv.conf file
```
touch /etc/resolv.conf
```
- Open the resolv.conf and add the following lines:
```
nameserver <IP Address of Host>

nameserver 1.1.1.1

search example.lan <- Change to domain
```
- Make /etc/resolv.conf immutable
```
chattr +i /etc/resolv.conf
```
## Install Samba 
```
apt update && apt upgrade -y
```
- Install Samba and all packages
```
apt install -y acl attr samba samba-dsdb-modules samba-vfs-modules smbclient winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config krb5-user dnsutils chrony net-tools
```
- In the prompts, enter the following:
1. First prompt enter the realm domain name (i.e. example.lan)
2. Server FQDN (i.e. dc1.example.lan)
3. FQDN to specify Kerberos administrative server (i.e. dc1.example.lan)
- Stop and disable services that Samba AD does not require including ```smbd```, ```nmbd```, and ```winbind```.
```
systemctl disable --now smbd nmbd winbind
```
- Finally, run and enable samba-ad-dc
```
systemctl unmask samba-ad-dc

systemctl enable samba-ad-dc
```
## Configuring Samba Active Directory
- Create backup of smb.conf
```
mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
```
Run the samba-tool
```
samba-tool domain provision
```
Enter the following:
1. Realm: [Enter default if none, enter example.lan]
2. Domain: [Default]
3. Server Role [dc]
4. DNS Backend [SAMBA_INTERNAL]
5. DNS Forwarder IP Address: 1.1.1.1
- Now run the commands to backup the default Kerberos configuration:
```
mv /etc/krb5.conf /etc/krb5.conf.bak

cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```
- Finally, execute the systemctl command to start the Samba Active Directory
```
systemctl start samba-ad-dc

systemctl status samb-ad-dc
```
## [Optional] Setup Time Synchronization
- Benefits of this include replay attack prevention.
```
chown root:_chrony /var/lib/samba/ntp_signd/

chmod 750 /var/lib/samba/ntp_signd/
```
- Open the /etc/chrony/chrony.conf and add the following configurations to the bottom of the file.
```
bindcmdaddress <IP of Host>

allow <CIDR Subnet>

ntpsigndsocket /var/lib/samba/ntp_signd
```
- Restart the chronyd service
```
systemctl restart chronyd

systemctl status chronyd
```
## Verify Samba Active Directory
- Run the host command
```
host -t A example.lan

host -t A dc1.example.lan
```
- Verify _kerberos and _ldap services are working
```
host -t SRV _kerberos._udp.example.lan

host -t SRV _ldap._tcp.example.lan
```
- Verify default resources available on Samba Active Directory
```
smbclient -L example.lan -N
```
- Lastly, run kinit to authenticate to Kerberos server using the User Administrator (Note: EXAMPLE.LAN must be uppercase).
```
kinit administrator@EXAMPLE.LAN

klist
```
## Creating a new Samba Active Directory User
- Run the following command to create a user:
```
samba-tool user create <username> <password>
```
- To verify that the user was created:
```
samba-tool user list
```
## Join and Logging In to Samba Active Directory Domain
- To begin, open PowerShell as Administrator
- Next, run the following command to list the ethernet adapters on Windows PC:
```
Get-NetAdapter -Name "*"
```
(*Note the adapter name*)
- After that, execute the following command to change the adapter's DNS server to point to the Samba's Active Directory IP address with 1.1.1.1 as a fallback DNS.
```
Set-DNSClientServerAddress "Ethernet" -ServerAddress ("<IP Address>","1.1.1.1")
```
- Run the following command to verify that the DNS resolver is pointing to the Samba AD IP address
```
Get-DnsClientServerAddress
```
- To verify that you can reach the Samba AD, run the following:
```
ping dc1.example.lan

ping example.lan
```
- To add the computer to the domain, run:
```
#this will add the computer to the domain "example.lan" and force a restart
Add-Computer -DomainName "example.lan" -Restart 
```
- When prompted, add the administrator username and password that was previously configured.
- Finally, go to other user and sign in as the newly created user in [Creating a new Samba Active Directory User](https://github.com/stevesec/active_directory#creating-a-new-samba-active-directory-user)
