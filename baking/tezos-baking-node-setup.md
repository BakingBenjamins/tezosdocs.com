---
description: How to compile Tezos from source and run node software in "screen" sessions.
---

# Setup Tezos Baking Node

Below you will find a low tech way to run a homebrew type of Tezos node. This guide is intended to be short, technical and with as little information as necessary to get the job done. You can reach out to us in [https://t.me/BakingBenjamins](https://t.me/BakingBenjamins) if more information is needed.

## Prepare to install Tezos node

This guide starts with the latest Ubuntu 20.10 server version but it should work on all Debian based distributions and older \(not ancient\) versions.

### Add non-root user \(for security\)

`adduser YOUR_username_here  
usermod -aG sudo YOUR_username_here`

This is your “non-root” username. Don't run things as root, it's not worth the problems.

### \(optional\) Setup ZeroTier \(for node and remote signer communication\) \[_run as root or sudo_\]

{% hint style="info" %}
_ZeroTier is a simple way to setup 2 machines to talk to each other securely._ Run on both remote signer and VPS machines. This puts them on the same network. Now you have a really simple and secure way to connect from the 2 servers.
{% endhint %}

`curl -s 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/master/doc/contact%40zerotier.com.gpg' | gpg --import && \  
if z=$(curl -s 'https://install.zerotier.com/' | gpg); then echo "$z" | sudo bash; fi  
sudo zerotier-cli join <YOUR_NETWORK_HERE>`

### Install operating system prerequisites \[_run as root or sudo_\]

`sudo apt install -y curl xz-utils jq screen build-essential git m4 unzip rsync curl bubblewrap libev-dev libgmp-dev pkg-config libhidapi-dev jbuilder software-properties-common opam autoconf libffi-dev  
sudo apt update`

### Download Tezos blockchain snapshot 

{% hint style="info" %}
You can can work on next step while this one runs
{% endhint %}

Use one of the sources below to obtain a full or rolling \(both work for baking\) snapshot of the Tezos blockchain. A rolling snapshot will get you started the fastest and is completely fine to use for baking nodes.

{% embed url="https://snapshots.tulip.tools/\#/" caption="Tulip Tools" %}

{% embed url="https://snapshots-tezos.giganode.io/" caption="Giganode" %}

{% embed url="https://xtz-shots.io/" caption="MIDL" %}

## Compile Tezos node & import snapshot

### Download Tezos source code, initialize opam and build Tezos binaries

`cd ~  
git clone https://gitlab.com/tezos/tezos.git  
cd tezos  
git checkout latest-release  
git rev-parse HEAD  
cd  
opam init  
# (answer yes to questions)  
opam update  
cd ~/tezos  
make build-deps   
eval $(opam env)  
make`

### Generate Tezos node identity

`./tezos-node identity generate`

### Import chain snapshot and verify its legitimacy

{% hint style="info" %}
The import of the snapshot will take a no more than an hour on most systems.  If you're importing a full snapshot, the process may take up to a full day if you're running on a slow VM or fanless system
{% endhint %}

`./tezos-node snapshot import /path/to/chain.full --block <ENTER_BLOCK_HASH_HERE>`

Replace the made up path to where your downloaded and uncompressed the Tezos chain snapshot. Navigate to any Tezos blockchain explorer like [https://tzstats.com](https://tzstats.com) and [https://tzkt.io](https://tzkt.io) and look up the block number referenced by the snapshot website. The block will have a hash \# associated with it, which you will need to copy into the `<ENTER_BLOCK_HASH_HERE>` portion. This verifies the integrity of the chain and saves you many hours of synchronization and waiting.

## Start Tezos node and baking executables

### Start node and baking executables \(on standalone setup\)

**Run First Time Only to Add Ledger Support and Import Baker’s Key**

First add udev rules then import ledger signer info.

`wget -q -O - https://raw.githubusercontent.com/LedgerHQ/udev-rules/709581c85db97bf6ea12e472aa4e350bf0eabfb7/add_udev_rules.sh | sudo bash  
udevadm trigger  
udevadm control --reload-rules  
./tezos-client list connected ledgers  
# (replace ledger://<4-words-here> commands below with YOUR unique ledger path)  
./tezos-signer import secret key baker "ledger://<4-words-here>/bip25519/0h/0h"  
# (run only first time; to have it available just in case for the other use case with 2 computers)  
./tezos-client import secret key baker "ledger://<4-words-here>/bip25519/0h/0h"  
# (run only first time)`

**Run Every Time: Start Ledger/Node/Baker/Endorser/Accuser Processes**

`screen -S TezosNode  
./tezos-node run --rpc-addr 127.0.0.1`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`./tezos-client setup ledger to bake for baker --main-hwm 1061796  
# (changes ledger to latest tezos block to prevent double baking)   
screen -S TezosBakerEdo  
export TEZOS_LOG='* -> debug'  
./tezos-baker-008-PtEdo2Zk run with local node ~/.tezos-node baker`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S TezosEndorserDelhi  
export TEZOS_LOG='* -> debug'  
./tezos-endorser-008-PtEdo2Zk run baker`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S TezosAccuserEdo  
export TEZOS_LOG='* -> debug'  
./tezos-accuser-008-PtEdo2Zk run`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

### Start node and baking executables \(on remote signer setup\)

#### Run on Ledger Machine \(Ledger attached here\)

**Run First Time Only to Add Ledger Support and Verify**

First add udev rules then import ledger signer info.

`wget -q -O - https://raw.githubusercontent.com/LedgerHQ/udev-rules/709581c85db97bf6ea12e472aa4e350bf0eabfb7/add_udev_rules.sh | sudo bash  
udevadm trigger  
udevadm control --reload-rules  
./tezos-client list connected ledgers`

**Run Every Time: Start Node Service and Verify Ledger Connectivity**

`screen -S TezosNode  
./tezos-node run --rpc-addr 127.0.0.1`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

**Run First Time Only to Import Ledger Info**

`./tezos-client list connected ledgers   
# (replace ledger://<4-words-here> commands below with YOUR unique ledger path)  
./tezos-signer import secret key baker "ledger://<4-words-here>/bip25519/0h/0h"   
# (run only first time)  
./tezos-client import secret key baker "ledger://<4-words-here>/bip25519/0h/0h"   
# (run only first time)`

**Run Every Time: Start Node Service**

`./tezos-client setup ledger to bake for baker --main-hwm 958888  
# (change to latest tezos block)  
screen -S Signer  
./tezos-signer launch socket signer -a X.X.X.X -p 22222`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

#### Run on VPS Machine \(Baker running here & connecting to remote Ledger machine\)

**Run First Time Only to Import Key**

`./tezos-client import secret key baker tcp://X.X.X.X:22222/tz1S5WxdZR5f9NzsPXhr7L9L1vrEb5spZFur`

**Run Every Time: Start node/baker/endorser/accuser/ping**

`screen -S TezosNode  
./tezos-node run --rpc-addr 127.0.0.1`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S TezosBakerEdo  
export TEZOS_LOG='* -> debug'  
./tezos-baker-008-PtEdo2Zk run with local node ~/.tezos-node baker`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S TezosEndorserEdo  
export TEZOS_LOG='* -> debug'  
./tezos-endorser-008-PtEdo2Zk run baker`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S TezosAccuserEdo  
export TEZOS_LOG='* -> debug'  
./tezos-accuser-008-PtEdo2Zk run`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S TezosPing  
ping -D -O -i 7 X.X.X.X`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}





> 🙏 _Donate & help us grow. All proceeds go to more baking capacity._  
>                                                        **tz1S5WxdZR5f9NzsPXhr7L9L1vrEb5spZFur**

