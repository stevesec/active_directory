# Deployment of Samba Active Directory

* Install Ubuntu Server on Proxmox PVE or Virtual Host
1. Set hostname 

```
hostnamectl set-hostname dc1
```

2. Open /etc/hosts with preferred text editor
 - Append this to the end of the hosts file

```
#change FQDN to whatever you would like 
<IP Address of Host> dc1.example.lan dc1

```

3. Verify FQDN
```
#verify FQDN
hostname -f

#verify FQDN is resolved to the Samba IP address
ping -c3 dc1.example.lan
```

4. Restart disable the DNS Resolver
Ubuntu DNS Configuration is not suitable for Samba Active Directory.

```
systemctl disable --now systemd-resolved


unlink /etc/resolv.conf
```

Then, create a new resolv.conf file

```
touch /etc/resolv.conf

```

Open the resolv.conf and add the following lines:

```
#Samba server IP address

nameserver <IP Address of Host>

# fallback resolver

nameserver 1.1.1.1

#main domain for Samba

search example.lan <- Change to domain

```
5. 