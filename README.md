# minecraftserv

Oracle offers a free tier for cloud instances running Ubuntu and Ampere A1 Arm OCPUs. We can set up, for free, multiple instances totalling 4 OCPUs and 24 GB of memory. We will set up an Ubuntu instance using 2 OCPUs and 8 GB RAM to run a Minecraft server using PaperMC.  

## 1. Sign up for Oracle Cloud Free Tier https://www.oracle.com/cloud/free/.  
This may be a process due to MFA being enabled from the start. Once set up, allow email MFA. 

## 2. Create a VM Instance - from main dashboard select Create a VM Instance
a) Name: mcservername; Compartment: leave default  
b) Placement: leave up to oracle defaults  
c) Security: ignore  
d) Image and Shape: Ubuntu 22.04 (latest might not permit AMD shape); Ampere VM.Standard.A1.Flex with 2 OCPUs and 8GB RAM;  
e) Primary VNIC Information: VCN and Subnet should be already filled in by default; Private and Public IPv4 addresses assigned should already be checked by default;  
f) SSH keys: generate key pair and download both private and public keys (or generate own using OpenSSH and upload public key file, but this did not work)  
g) Boot volume: uncheck all options  
h) Click "Create"; if system cannot create then save stack and re-try; can use a script or upgrade to pay-as-you-go tier (whilst still using only free resources)  
i) If successful, it will show that instance is 'provisioning'. When complete, copy the public IP address to access it.  

## 3. Connect to IP address using ssh  
a) 
```
$ ssh ubuntu@IP.ADD.RESS  	['ubuntu' is default user for this setup?]
```
b) If get error such as 'permissions', copy private key file to user-specific directory and retry. System checks how available key file is so copy it to user-specific directory.   
```
$ ssh -i C:\Users\USER\Documents\ssh-key.key ubuntu@IPADDRESS  
```
[*Optional: In OCI management console, update inbound rules to allow ssh access only from your own IP address.]  

## 4. Create secondary user to ssh with  
a) 
```
$ sudo adduser testuser1		[appears /home directory is added automatically]
```
b) 
```
$ groups ubuntu                  	[to list groups default user is part of]
```
c) 
```
$ sudo usermod -aG sudo testuser1	[to add testuser1 to sudo group]
```
d) Permit ssh access by modifying .conf file in /etc/ssh/sshd_config.d/ directory  
```
$ nano filenamewithsettings.conf
```
[add the following four lines to .conf file]
```
Match User <username>  
PasswordAuthentication yes  
Match all  
PasswordAuthentication no
```
e) Restart ssh service		[open secondary terminal to test connection]
```
$ sudo systemctl restart ssh.service
```
5. Repeat all above to create second instance (use for different server or test server?)
mcsrvetcetc
1 OCPU
8 GB Ram







### Sources
https://blogs.oracle.com/developers/post/how-to-set-up-and-run-a-really-powerful-free-minecraft-server-in-the-cloud  
https://github.com/mgrimace/Minecraft-on-Oracle  
https://tuxinit.com/host-paper-minecraft-server-in-linux/  

### Information on free tier resources from oracle
https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm  
https://docs.oracle.com/en-us/iaas/Content/Quotas/Concepts/resourcequotas.htm  

### Other
https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement
https://www.reddit.com/r/oraclecloud/
https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys
