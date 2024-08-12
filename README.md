# minecraftserv

Oracle offers a free tier for cloud instances running Ubuntu and Ampere A1 Arm OCPUs. We can set up, for free, multiple instances totalling 4 OCPUs and 24 GB of memory. We will set up an Ubuntu instance using 2 OCPUs and 8 GB RAM to run a Minecraft server using PaperMC.  

## Setting up cloud instance

### 1. Sign up for Oracle Cloud Free Tier
Go to https://www.oracle.com/cloud/free/ to start. This may be a process due to MFA being enabled from the start. Once set up, allow email MFA. 

### 2. Create a VM Instance - from main dashboard select Create a VM Instance
a) Name: mcservername; Compartment: leave default  
b) Placement: leave up to oracle defaults  
c) Security: ignore  
d) Image and Shape: Ubuntu 22.04 (latest might not permit AMD shape); Ampere VM.Standard.A1.Flex with 2 OCPUs and 8GB RAM;  
e) Primary VNIC Information: VCN and Subnet should be already filled in by default; Private and Public IPv4 addresses assigned should already be checked by default;  
f) SSH keys: generate key pair and download both private and public keys (or generate own using OpenSSH and upload public key file, but this did not work)  
g) Boot volume: uncheck all options  
h) Click "Create"; if system cannot create then save stack and re-try; can use a script or upgrade to pay-as-you-go tier (whilst still using only free resources)  
i) If successful, it will show that instance is 'provisioning'. When complete, copy the public IP address to access it.  

### 3. Connect to IP address using ssh  
a) 
```
$ ssh ubuntu@IP.ADD.RESS  	['ubuntu' is default user for this setup?]
```
b) If get error such as 'permissions', copy private key file to user-specific directory and retry. System checks how available key file is so copy it to user-specific directory.   
```
$ ssh -i C:\Users\USER\Documents\ssh-key.key ubuntu@IPADDRESS  
```
[*Optional: In OCI management console, update inbound rules to allow ssh access only from your own IP address.]  

### 4. Create secondary user to ssh with  
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
### 5. Repeat all above to create second instance (use for different server or test server?)
mcsrvetcetc
1 OCPU
8 GB Ram

## Setting up server 
We will set up and configure PaperMC as our minecraft server.  

### 1. Log into ubuntu instance and update OS
```
$ sudo apt-get update
$ sudo apt-get upgrade
```

### 2. Install wget and tmux
```
$ sudo apt-get install wget
$ sudo apt-get install tmux
```
### 3. Create new user to manage server [optional]
```
$ sudo useradd -m -s /bin/bash papercraft
$ sudo su - papercraft
```
### 4. Download PaperMC server .jar file
```
$ cd ~ && mkdir papermc && cd papermc
```
Check website (https://papermc.io/downloads/paper) for latest .jar file and use that link below:
```
$ wget https://api.papermc.io/v2/projects/paper/versions/1.21.1/builds/9/downloads/paper-1.21.1-9.jar
```

### 5. Install java  
a) From: https://docs.papermc.io/misc/java-install  
```
$ sudo apt-get update && sudo apt-get upgrade
$ sudo apt-get install ca-certificates apt-transport-https gnupg wget
```
b) Import repository for Amazon Coretto java OPENJDK distribution
```
$ wget -O - https://apt.corretto.aws/corretto.key | sudo gpg --dearmor -o /usr/share/keyrings/corretto-keyring.gpg && \
echo "deb [signed-by=/usr/share/keyrings/corretto-keyring.gpg] https://apt.corretto.aws stable main" | sudo tee /etc/apt/sources.list.d/corretto.list
```
c) Install Java 21 and other dependencies
```
$ sudo apt-get update
$ sudo apt-get install -y java-21-amazon-corretto-jdk libxi6 libxtst6 libxrender1
```
d) Verify installation
```
$ java -version
```
### 6. 
a) Set up startup script using https://docs.papermc.io/misc/tools/start-script-gen
-max memory usage 6GB
-nogui
-adjust "server.jar" file to the papermc one downloaded previously

b) Copy .sh file to cloud instance using scp:
```
$ scp -i C:\Users\USER\Docs\ssh-key.key start.sh ubuntu@IP.AddR.ESS:"/home/ubuntu/papermc/"
```
c) Make script executable
```
$ chmod +x filename.sh
```
d) Run script
```
$ tmux
$ ./start.sh
```
e) Edit eula.txt to set eula=true

### 7. Server configuration
a) Edit server.properties 	[more details found at https://minecraft.wiki/w/Server.properties]
```
difficulty=normal
gamemode=survival 	[or creative]
query.port=25555	[choose port here]
rcon.port=25555		[choose port here]
server-port=25555	[choose port here]
online-mode=false
pvp=false
white-list=true
level-name=mineworld	[enter any name or leave default]
level-seed=		[leave blank for random seed]
```
Run start.sh again to initialize and create /config folders.

### 8. Global and per-world configuration
a) Edit config/paper-world-defaults.yml		[https://docs.papermc.io/paper/vanilla]  
b) Edit config/paper-global.yml			[https://docs.papermc.io/paper/vanilla]  
c) Edit spigot.yml				[https://docs.papermc.io/paper/vanilla]  
*Note: Ignored spigot.yml and left it at defaults. 

### 9. Adding plugins:
https://docs.papermc.io/paper/adding-plugins  

a) Add Coreprotect plugin to allow for rollback changes (in case of griefing):  
https://www.spigotmc.org/resources/coreprotect.8631/  
i) stop server by typing 'stop' in console  
ii) download .jar file and copy to plugins folder in server install directory  
iii) restart server  

### 10. Starting server with tmux:
```
$ tmux new -s mysessionname  
$ ./start.sh  
Ctrl +b  d  
```
Basic tmux commands:  
```
tmux ls 	[list sessions]  
tmux		[start session]  
tmux new -s <session-name>	[start session with specific name]  
tmux a -t mysession	[enter active session]  
Ctrl + b  d 		[detach from session]  
```

### 11. Adjust firewall options on cloud instance to allow access to server
a) Go to instance information in cloud account; in Primary VNIC section click on 'Subnet'  
b) Click on default security list  
c) Click on Add Ingress Rules  
d) Add 2 Ingress Rules - one for TCP and one for UDP - each with a ’Source CIDR’ of 0.0.0.0/0 and a destination port range of [YOURPORT from earlier, such as 25555].  
e) In ssh session, run the following:  
```
$ sudo sudo ufw allow 25555/tcp
$ sudo sudo ufw allow 25555/udp
$ sudo ufw status
$ sudo ufw enable	[if disabled]
```

*Initially did not work until rebooted instance.
Check IP using:  
```
$ host myip.opendns.com resolver1.opendns.com
```

### 12. Add names to whitelist.json using server console
```
whitelist add user1
whitelist add user2
whitelist list
whitelist remove user3
whitelist on/off 
```
*This does not secure the server. 


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

### PaperMC  
https://docs.papermc.io/paper/getting-started  

