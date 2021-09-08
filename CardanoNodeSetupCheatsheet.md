
<snippet>
  <content><![CDATA[
  
![Logo](https://github.com/granada-pool/ADA/blob/main/l_64.png) By GranADA_Staking_Pool [GRANA]

# Setup Cheatshet for Cardano nodes


I’ve compiled this cheatsheet to setup Cardano nodes (Relays and BPs) using cntools. This document also includes commands needed to successfully configure version 1.29.0 for the Alonzo hard fork (provided by [CHRTY]). Aside from my personal scripts to enhance the security in my servers, I’ve used the excellent content provided by staking pool operators of our community as source of inspiration.

Hope you find this useful and don’t hesitate to contact me if you need any help or have any questions!

Best regards,

Vito Melchionna

SPO of GranADA_Staking_Pool [GRANA]

granada.staking.pool@gmail.com






## Content

[Resources](https://github.com/granada-pool/ADA/new/main#resources)

[First steps](https://github.com/granada-pool/ADA/new/main#first-steps)

[Creating a non-root user](https://github.com/granada-pool/ADA/new/main#creating-a-non-root-user)

[Update Ubuntu](https://github.com/granada-pool/ADA/new/main#update-ubuntu)

[Disable Root Login/Set New SSH Port](https://github.com/granada-pool/ADA/new/main#disable-root-loginset-new-ssh-port)

[Server backups (snapshot in private server)](https://github.com/granada-pool/ADA/new/main#server-backups-snapshot-in-private-server)

[Creating firewall](https://github.com/granada-pool/ADA/new/main#creating-firewall)

[Disable Wifi and Bluetooth (Hardware server)](https://github.com/granada-pool/ADA/new/main#disable-wifi-and-bluetooth-hardware-server)

[Disable sleep, hibernation and ping command](https://github.com/granada-pool/ADA/new/main#disable-sleep-hibernation-and-ping-command)

[Configure SWAP for RAM](https://github.com/granada-pool/ADA/new/main#configure-swap-for-ram)

[Security (Google 2FA & Fail2Ban)](https://github.com/granada-pool/ADA/new/main#security-google-2fa--fail2ban)

[Synchonisation (Chrony)](https://github.com/granada-pool/ADA/new/main#synchonisation-chrony)

[Installing Prereqs](https://github.com/granada-pool/ADA/new/main#installing-prereqs)

[Installing cardano-node and cardano-cli](https://github.com/granada-pool/ADA/new/main#installing-cardano-node-and-cardano-cli)

[Get mainnet-alonzo-genesis.json file](https://github.com/granada-pool/ADA/new/main#get-mainnet-alonzo-genesisjson-file)

[Configuring CNODE script to use all available CPU cores (reduces missing slots)](https://github.com/granada-pool/ADA/new/main#configuring-cnode-script-to-use-all-available-cpu-cores-reduces-missing-slots)

[Configuring Env and Starting Node](https://github.com/granada-pool/ADA/new/main#configuring-env-and-starting-node)

[Editing topologyUpdater.sh For Relays](https://github.com/granada-pool/ADA/new/main#editing-topologyupdatersh-for-relays)

[Editing topology.json for Producer](https://github.com/granada-pool/ADA/new/main#editing-topologyjson-for-producer)

[Creating wallet/Registering pool](https://github.com/granada-pool/ADA/new/main#creating-walletregistering-pool)

[Monitoring (Grafana)](https://github.com/granada-pool/ADA/new/main#monitoring-grafana)

[Miscellaneous commands](https://github.com/granada-pool/ADA/new/main#miscellaneous-commands)

## Resources

[CNtools](https://cardano-community.github.io/guild-operators/Scripts/cntools/) [AAA] [CLIO1] [RDLRT] [UNDR] [AHL] [LOVE] [AEON] [BCSH] [EDEN] [STR8] [SMAUG] [PEGA]

[Setup tutorial video](https://www.youtube.com/watch?app=desktop&v=aEBMVB7qQC8) [PHNIX]

[Security video 2FA](https://www.youtube.com/watch?app=desktop&v=ga7gmZIS_0Y) [EDEN]

[Synchronisation video Chrony](https://www.youtube.com/watch?app=desktop&v=bkEz7GZLFVY) [EDEN]

[Configuring Grafana](https://www.youtube.com/watch?app=desktop&v=lcIWj1Xm3UM) [EDEN]

[General setup guide](https://github.com/Chris-Graffagnino/Jormungandr-for-Newbs/blob/master/docs/jormungandr_node_setup_guide.md) [MASTR]

## First steps

•	Create Droplet on data center or install Ubuntu on private server (16 GB RAM for v1.29.0)

•	Generate SSH key with PuTTY Keygen (RSA 4096) and save it in your cold storage

•	Store credentials and SSH passwords on password manager

`Note: Remember to change all [fields in square brackets]`

## Creating a non-root user
```html
sudo adduser [username]

sudo adduser [username] sudo

sudo su - [username]

mkdir ~/.ssh

cd .ssh
```

(Login with PuTTY as new user with new SSH key)

```html
nano ~/.ssh/authorized_keys 
```
(Paste SSH public key -> version displayed by PuTTY keygen)


## Update Ubuntu
```html
sudo apt-get update
sudo apt-get upgrade
```

## Disable Root Login/Set New SSH Port

`sudo nano /etc/ssh/sshd_config`
>	uncomment port and add number & “PermitRootLogin no” “PasswordAuthentication no”
```html
sudo systemctl restart ssh

sudo systemctl status ssh 
```
> test that login in with root is no longer possible. Add firewall rules for droplets

## Server backups (snapshot in private server)

If you are running private servers to enhance your pool’s decentralization, you can create snapshots using any of these 2 options:

*1. TimeShift (GUI)*

`sudo apt-get install timeshift`

Make sure that your physical volume is formatted in ext4

```html
sudo umount /dev/sdb1
sudo mkfs.ext4 -f /dev/sdb1

sudo timeshift --create --comments [snapshot’s name] --tags D --snapshot-device [target physical volume]

```
>	This command will create a snapshot of the server and a new config file at this location: /etc/timeshift.json
to restore
`sudo timeshift --list`

to restore from a snapshot
`sudo timeshift --restore --snapshot [snapshot’s name]`

delete a snapshot
`sudo timeshift --delete  --snapshot [snapshot’s name]`

*2. LVM (Recommended)*

>LVM can be implemented in several ways, so make sure you are aware how your physical volumes and partitions are organized before following this guide.

Check if there are any volume groups available

`sudo vgdisplay`

You can either configure a LVM partition on a physical volume when installing Ubuntu or just configure a USB stick (32GB) with a LVM partition. First check what volumes are available and select the one to set the LVM partition

`sudo fdisk -l`

Check if there are any volume groups available

`sudo vgdisplay`

We will assume that we will create a new partition volume in dev/sdb1 (external drive)

```html
#This will wipe out all the data in your USB stick and will write the LVM header to the partition. Just make sure this device doesn’t have any important data and answer “y” to all prompts
sudo pvcreate /dev/sdb1


#Create volume group
sudo vgcreate [vg name] /dev/sdb1

#Create logical volume
sudo lvcreate -n [lv name] -L 32g [vg name] 

#Create snapshot
sudo lvcreate -s -n [snapshot name] -L 32g [vg name]/[lv name]

#Merge snapshot into the origin volume for recovering
sudo lvconvert --merge [vg name]/[snapshot name] 
```

Other useful commands

```html

#Resizing partitions
sudo lvextend -L +5g [vg name]/[lv name]

#Expand filesystem (after resizing)
sudo resize2fs /dev/[vg name]/[lv name]

#Moving partitions
sudo pvmove -n [lv name] /dev/sdb1

#Remove logical volume
sudo lvremove [vg name]/[lv name]

```

## Creating firewall

```html
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw limit proto tcp from any to any port [custom ssh port]
sudo ufw limit proto tcp from any to any port [Cardano port for node]
```
*For Relay Nodes*

`sudo ufw allow [relay port]/tcp`

*For Producer Node*
```html
sudo ufw allow proto tcp from [relay 1 ip] to any port [producer port]
sudo ufw allow proto tcp from [relay 2 ip] to any port [producer port]
```

*Finally (both)*
```html
sudo ufw enable
sudo systemctl restart ssh
sudo ufw status 
```
## Disable Wifi and Bluetooth (Hardware server)

```html
systemctl stop bluetooth
systemctl disable bluetooth.service
nmcli radio wifi off
```

## Disable sleep, hibernation and ping command

```html
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
sudo sysctl -w net.ipv4.icmp_echo_ignore_all=1
```
## Configure SWAP for RAM

```html
# Swap utilizes diskspace to temporarily handle spikes in memory usage
# Skip this section if you have limited diskspace, (you're running a raspberry-pi, for instance).

# Show current swap configuration
sudo swapon --show

# Check what swap is currently active, if any
free -h

# Check current disk usage
df -h

# Create swap file (Don't forget the "G")
sudo fallocate -l [SIZE =< TO RAM]G /swapfile

# Verify swap settings
ls -lh /swapfile

# Only root can access swapfile
sudo chmod 600 /swapfile

# Mark the file as swap space
sudo mkswap /swapfile

# Enable swap settings every time we log in
# Make a backup of /etc/fstab
sudo cp /etc/fstab /etc/fstab.bak

# Type this command from the command-line to add swap settings to the end of fstab
echo '/swapfile none swap sw 00' | sudo tee -a /etc/fstab

# Enable swap
sudo swapon -a

# Verify swap is enabled
free -h
```
## Security (Google 2FA & Fail2Ban)

```html
# Checks logs for ssh logins
sudo journalctl -u ssh 


# Checks logs for ssh logins with more details
sudo journalctl -eu ssh

sudo apt install libpam-google-authenticator
# y to all questions
google-authenticator 
sudo nano /etc/ssh/sshd_config
```
>ChallengeResponseAuthentication yes | usePAM yes

```html
sudo systemctl restart ssh
sudo nano /etc/pam.d/sshd
```
>Include: 
#One-time authentication vie Google Authenticator
auth required pam_google_authenticator.so

`sudo nano /etc/ssh/sshd_config`

>Include (at the end of file): 
AuthenticationMethods publickey,keyboard-interactive

```html
sudo systemctl restart ssh
sudo apt install fail2ban
sudo systemctl status fail2ban

#checks banned IPs
sudo iptables -L | grep f2b

#if the config file must be changed (defaults are OK)
cd /etc/fail2ban
sudo nano jail.conf
```
## Synchronisation (Chrony)

```html
sudo apt-get install chrony
systemctl status chrony
sudo nano /etc/chrony/chrony.conf
```
>Replace: 
```html
pool time.google.com       iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool time.facebook.com     iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool time.euro.apple.com   iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool time.apple.com        iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3
pool ntp.ubuntu.com        iburst minpoll 2 maxpoll 2 maxsources 3 maxdelay 0.3

maxupdateskew 5.0

makestep 0.1 -1

#GET TAI-UTC offset and leap seconds from the system tz database
leapsectz right/UTC
```
>Include: 
```html
# Serve time even if not synchronized to a time source
local stratum 10
```
`sudo systemctl restart chrony`

## Installing Prereqs
```html
mkdir "$HOME/tmp"
cd "$HOME/tmp"
curl -sS -o prereqs.sh https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/prereqs.sh

chmod 755 prereqs.sh
./prereqs.sh -f
. "${HOME}/.bashrc"
```
## Installing cardano-node and cardano-cli
```html
cd ~/git
git clone https://github.com/input-output-hk/cardano-node
cd cardano-node
git fetch --tags --all
git pull
git checkout $(curl -s https://api.github.com/repos/input-output-hk/cardano-node/releases/latest | jq -r .tag_name)

$CNODE_HOME/scripts/cabal-build-all.sh -o

#test installation
cardano-cli version
cardano-node version
```
## Get mainnet-alonzo-genesis.json file

```html
cd $CNODE_HOME/files
curl -sL -f -o alonzo-genesis.json https://hydra.iohk.io/job/Cardano/iohk-nix/cardano-deployment/latest-finished/download/1/mainnet-alonzo-genesis.json

chmod 755 alonzo-genesis.json
ls -l
nano config.json
```
>Include (on the top):
```html
"AlonzoGenesisFile": "/opt/cardano/cnode/files/alonzo-genesis.json",
"AlonzoGenesisHash": "7e94a15f55d1e82d10f09203fa1d40f8eede58fd8066542cf6566008068ed874"
```
`sudo systemctl restart cnode`

## Configuring CNODE script to use all available CPU cores (reduces missing slots)
```html
cd $CNODE_HOME/scripts
lscpu 
#remember number of property “CPU(s)” -> available cores
sudo nano cnode.sh
#uncomment CPU variable and set it to max number of available cores
#save changes and restart node (this avoids missing slots)
```
>Having a BP on a bare metal dedicated server and turning off the TraceMempool property on the config.json file also helps to avoid this issue

## Configuring Env and Starting Node

```html
cd $CNODE_HOME/scripts
nano env
```
Change line with CNODE_PORT to:

`CNODE_PORT=[DESIRED NODE PORT]`

Press Ctrl + X to exit

Press Y to save modified buffer

Press Enter to keep file name

```html
./deploy-as-systemd.sh 
[ When Asked About topologyUpdater Select Y for Relays and N for Producer ]
sudo systemctl start cnode.service
sudo systemctl status cnode.service
./gLiveView.sh #OR ./sLiveView.sh
```

## Editing topologyUpdater.sh For Relays

```html
cd $CNODE_HOME/scripts
nano topologyUpdater.sh
```
Delete # from line with CUSTOM_PEERS and change to:

`CUSTOM_PEERS = "producer ip:port|relays-new.cardano-mainnet.iohk.io:3001:2"`

Press Ctrl + X to exit

Press Y to save modified buffer 

Press Enter to keep file name

`sudo systemctl restart cnode.service`

## Editing topology.json for Producer
```html
cd $CNODE_HOME/files
nano topology.json
```
Delete Everything and add:
```html
{
  "Producers": [
    {
      "addr": "relay 1 ip",
      "port": relay port,
      "valency": 1
    },
    {
      "addr": "relay 2 ip",
      "port": other relay port,
      "valency": 1
    }
  ]
}
```
Press Ctrl + X to exit

Press Y to save modified buffer

Press Enter to keep file name

`sudo systemctl restart cnode.service`

## Creating wallet/Registering pool

`./cntools.sh`

##	Monitoring (Grafana)

*Relay (host) part I*
```html
cd $CNODE_HOME/scripts
./setup_mon.sh #setup script from cntools
#open ports for Prometheus, EKG & Grafana
sudo ufw allow proto tcp from 127.0.0.1 to any port 9090 
sudo ufw allow proto tcp from 127.0.0.1 to any port 9091 
sudo ufw allow proto tcp from 127.0.0.1 to any port 12798
sudo ufw allow proto tcp from any to any port 5000

#Now you can check the Grafana dashboard in a browser (PC has to be connected to local network) -> [IP|DNS]:5000
#Check port forwarding in router in case that this doesn’t work
#Get JSON config and copy it from this link
#On Grafana Client: Dashboards->manage->add new panel(button)->dashboard settings (icon)-> JSON Model->Paste JSON config and save
```

*BP*
```html
cd $CNODE_HOME/files
sudo nano config.json 
#change hasPrometheus property (ip: 0.0.0.0) and save -> restart node

sudo systemctl restart cnode.service
cd
cd tmp
wget https://raw.githubusercontent.com/DamjanOstrelic/Cardano-stuff/master/setup_node_exporter.sh
chmod 755 setup_node_exporter.sh
./setup_node_exporter.sh
sudo ufw allow proto tcp from [IP host] to any port 9091 
sudo ufw allow proto tcp from [IP host] to any port 12798
netstat -tulpn # test that the ports are open and being listened
```
*Relay (host) part II*
```html
cd $CNODE_HOME
cd ..
cd monitoring
cd Prometheus
sudo nano prometheus.yml 
# Add in scrape_configs
  - job_name: 'VIMinerII_cardano_node'
    static_configs:
    - targets: ['[Local IP BP]:12798']
      labels:
        instance: "VIMinerII_cnode"
  - job_name: '[BP_NAME]_node_exporter'
    static_configs:
    - targets: ['[Local IP BP]:9091']
      labels:
        instance: "[BP_NAME]_node_exporter"

sudo systemctl restart Prometheus
sudo systemctl status prometheus
sudo systemctl restart node_exporter.service
```
## Miscellaneous commands
*Check for missing slots*
`curl localhost:12798/metrics | grep "cardano_node_metrics_slotsMissedNum_int"`

*Logging (Debug)*
`journalctl -e -f -u cnode.service`


</content>
</snippet>

