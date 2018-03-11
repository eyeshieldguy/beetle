## Beetle Masternode Howto v1.03

How to setup a masternode with a Cold Wallet Setup – meaning the coins remain on your desktop wallet and you remote control the masternode on a linux VPS.

#### Outline of the steps required:
---
###### A) Get a VPS from a provider like DigitalOcean, Vultr, Linode, etc. 
   - Recommended VPS size at least 1gb RAM 
   - **It must be Ubuntu 16.04 (Xenial)**
   - Install required dependencies on the VPS
###### B) Clone Beetle Repo from github & Compile Wallet
###### C) Configure wallet for masternode
###### D) Configure Desktop wallet to remote control masternode
###### E) Check if masternode is running on VPS successfully
###### F) Install Firewall on VPS (IMPORTANT)

## A) VPS Setup
---
1. Log into your VPS

2. Copy/paste these commands into the VPS and hit enter: (One Box At A Time)
```
apt-get -y update
```
```
apt-get -y upgrade
```
```
apt-get -y install software-properties-common
```
```
apt-add-repository -y ppa:bitcoin/bitcoin
```
```
apt-get -y update
```
```
apt-get -y install \
    build-essential \
    libtool \
    autotools-dev \
    automake \
    pkg-config \
    libssl-dev \
    libevent-dev \
    bsdmainutils \
    libboost-all-dev \
    libdb4.8-dev \
    libdb4.8++-dev \
    libminiupnpc-dev \
    libzmq3-dev \
    git \
    nano \
    tmux \
    libgmp3-dev \
    wget
```

Enter the commands below to create a SWAP file of 2GB:
```
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
sysctl vm.swappiness=10
sysctl vm.vfs_cache_pressure=50
echo 'vm.swappiness=10' | tee -a /etc/sysctl.conf
echo 'vm.vfs_cache_pressure=50' | tee -a /etc/sysctl.conf
```
## B) Clone Beetle Repo from github & Compile Wallet
---
1. Clone the Beetle github repo:
```
git clone https://github.com/beetledev/BeetleCoin
```
2. Compile the wallet:
```
cd BeetleCoin/src/leveldb && chmod 777 * && cd .. && make -f makefile.unix
```
3. Configure crontab to start masternode if VPS reboots
```
crontab -e
```
Then add the following line at the bottom
```
@reboot /root/BeetleCoin/src/beetled
```
Exit & Save file by doing CTRL X to save it. Y for yes, then ENTER.

## C) Configure Wallet for masternode
---
1. Edit the configuration file:
```
mkdir /root/.Beetle && nano /root/.Beetle/Beetle.conf
```
2. Add following lines:
```
rpcuser=SOME_RANDOM_USERNAME
rpcpassword=SOME_RANDOM_PASSWORD
rpcallowip=127.0.0.1
listen=1
server=1
daemon=1
```
3. Exit & Save file by doing CTRL X to save it. Y for yes, then ENTER.
```
cd /root/BeetleCoin/src
```
4. Start the wallet daemon:
```
./beetled &
```
5. Generate Masternode Private Key
```
./beetled masternode genkey
```
You should see a long key that looks like:
```
3HaYBVUCYjEMeeH1Y4sBGLALQZE1Yc1K64xiqgX37tGBDQL8Xg
```
This is your `masternode private key`, keep it safe, do not share with anyone.
6. Then we stop the wallet:
```
./beetled stop
```
7. Add the private key to configuration file:
```
nano /root/.Beetle/Beetle.conf
```
8. Add masternode information:
Replace IP_ADDRESS_OF_VPS with your VPS ip address
Replace MASTERNODE_PRIVATE_KEY with the long key we got in step C.5
```
rpcuser=SOME_RANDOM_USERNAME
rpcpassword=SOME_RANDOM_PASSWORD
rpcallowip=127.0.0.1
listen=1
server=1
daemon=1
stake=0
port=45823
maxconnections=256
masternode=1
masternodeaddr=IP_ADDRESS_OF_VPS:45823
masternodeprivkey=MASTERNODE_PRIVATE_KEY
```
9. Run the wallet daemon (it will stay running in the background)
```
./beetled
```

### D) Configure Desktop wallet to remote control masternode.
---
1. Open the Beetle Desktop Wallet
2. Go to `RECEIVE` tab and click `New Address` and Label: `MN1`, then `OK`.
![Alt text](https://dils.biz/images/new_receive_address.png "New Address")
3. Right-Click on the Label `MN1` and click `Copy Address`.
4. Go to `SEND` tab and right-click in `Pay To:` and paste the address.
5. Amount should be `50000` BEETLE.  
6. Click `Send`.
7. A `Confirm send coins` screen will pop-up and you click `Yes` to make the transfer.
![Alt text](https://dils.biz/images/sendcoins.png "Send coins")
8. Wait for 15 confirmations on the transaction.  To check you can go to `TRANSACTIONS` and wait for a tick to appear on the left hand side of the transaction.
9. Go to `Tools` -> "Debug console"
10. Run the following command: `masternode outputs`
![Alt text](https://dils.biz/images/console.png "Console")
11. You should see output like the following if you have a transaction with exactly 50,000 BEETLE:
```
{
    "12345678xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx": "0"
}
```
12. The value on the left is your `txid` and the right is the `vout`
13. Go to the `Masternodes` tab
14. ![Alt text](https://dils.biz/images/masternode.png "Masternodes")
14. Click `Create` button and fill the details:
```
Alias: [Enter a name for your masternode, example MN1]
Address: [VPS_IP_ADDRESS:45823]
PrivKey:[MASTERNODE_PRIVATE_KEY created in Step C5 above]
TxHash:[txid from Step D11]
Output Index: [vout from Step D11]
Reward Address: [Leave empty]
Rewards %: [Leave empty]
```
![Alt text](https://dils.biz/images/AddNode.png "Add Node")
15. Click `OK` to add the masternode.
16. Click `Start All`.

### E) Check if masternode is running on VPS successfully

Go on the VPS and type:
```
./beetled masternode status
```
The output should be like below:
![Alt text](https://dils.biz/images/masternode_status1.png "masternode status")

Your VPS is running fine if  `"status" : 9` otherwise if it is not `9` go to your Desktop Wallet, to the `MASTERNODES`, select your masternode from the list and Click on `Start` again.

Wait for the masternode to mature and you will enjoy nice rewards very soon!

### F) Install Firewall on VPS (IMPORTANT)
Enter the commands below to install and setup the firewall:
(Copy/Paste each line in the VPS terminal)
```
apt-get -y install ufw
```
```
ufw default deny incoming
```
```
ufw default allow outgoing
```
```
ufw allow ssh
```
```
ufw allow 45823/tcp
```
```
ufw enable
```

### Enjoy your new VPS!  Cheers ! :)

---------
eyeshield
