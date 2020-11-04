---
description: >-
  How to compile Tezos from source & what other options exist to run a Tezos
  baking node.
---

# Tezos Baking Node Setup \(wip\)

{% hint style="info" %}
_Currently this guide is our own reference for setting up Tezos nodes. It is intended to be short, technical and with as little information as necessary to get the job done. You can reach out to us in_ [_https://t.me/BakingBenjamins_](https://t.me/BakingBenjamins) _if more information is needed. We are aware this is a very low tech way of running a Tezos node but it should give you a good understanding of all the steps you can automate any way to suit your preferences._
{% endhint %}

## Installing and Setting up Node

### Add Non-root User and allow to execute SUDO commands

`adduser YOUR_username_here  
usermod -aG sudo YOUR_username_here`  


This is your “non-root” username. Don't run things as root, it's not worth the problems.

### Setup ZeroTier \(for Node and Remote Signer communication\) \[_run as root_\]

{% hint style="info" %}
_ZeroTier is a simple way to setup 2 machines to talk to each other securely. You can use a variety of other methods; this guide will include some of them in the future_
{% endhint %}

`curl -s 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/master/doc/contact%40zerotier.com.gpg' | gpg --import && \  
if z=$(curl -s 'https://install.zerotier.com/' | gpg); then echo "$z" | sudo bash; fi  
zerotier-cli join <YOUR_NETWORK_HERE>`  


Run on both remote signer and VPS machines. This puts them on the same network.

### Install Prerequisites \[_run as root_\]

`apt install -y curl xz-utils jq screen build-essential git m4 unzip rsync curl bubblewrap libev-dev libgmp-dev pkg-config libhidapi-dev jbuilder software-properties-common opam  
add-apt-repository ppa:avsm/ppa  
apt update`

### Download Full Chain Snapshot \[as non-root\] \[can work on next step while this one runs\]

`cd ~  
curl -s https://api.github.com/repos/Phlogi/tezos-snapshots/releases/latest | jq -r ".assets[] | select(.name) | .browser_download_url" | grep full | xargs wget -q --show-progress  
cat mainnet.full.* | xz -d -v -T0 > mainnet.full`

Another option is to use the Tulip Tools snapshot, which may be newer.  You always want to make sure to use the latest available snapshot to minimize the sync time for your node when it first runs.

{% embed url="https://snapshots.tulip.tools/\#/" caption="Tulip Tools" %}

{% embed url="https://snapshots-tezos.giganode.io/" caption="Giganode" %}

{% embed url="https://xtz-shots.io/" caption="MIDL" %}

### Pull Tezos node source, install OPAM for OCaml and build node \[as non-root\]

`cd ~  
git clone`[ `https://gitlab.com/tezos/tezos.git`](https://gitlab.com/tezos/tezos.git) `&& cd tezos  
git checkout latest-release  
git rev-parse HEAD  
cd  
opam init         < — (answer yes to questions)  
opam update  
eval $(opam env)  
cd ~/tezos  
make build-deps   
eval $(opam env)  
make`

### Generate Tezos Node Identity \[as non-root\]

`./tezos-node identity generate`

### Import Full Chain Snapshot and Verify it’s Legitimate

`./tezos-node snapshot import ../mainnet.full --block <ENTER_BLOCK_HASH_HERE>`  


### Start Node and Start Baking

#### Run on Ledger Machine \(Ledger attached here\)

**Run First Time Only to Add Ledger Support and Verify**

First add udev rules then import ledger signer info.

`wget -q -O - https://raw.githubusercontent.com/LedgerHQ/udev-rules/709581c85db97bf6ea12e472aa4e350bf0eabfb7/add_udev_rules.sh | sudo bash  
udevadm trigger  
udevadm control --reload-rules  
./tezos-client list connected ledgers`

**Run Every Time: Start Node Service and Verify Ledger Connectivity**

`screen -S Node  
./tezos-node run --rpc-addr 127.0.0.1`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

**Run First Time Only to Import Ledger Info**

`./tezos-client list connected ledgers    < — (replace ledger://<4-words-here> commands below with YOUR unique ledger path)  
./tezos-signer import secret key baker "ledger://<4-words-here>/bip25519/0h/0h"    < — (run only first time)  
./tezos-client import secret key baker "ledger://<4-words-here>/bip25519/0h/0h"    < — (run only first time)`

**Run Every Time: Start Node Service**

`./tezos-client setup ledger to bake for baker --main-hwm 958888    < — (change to latest tezos block)  
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

`screen -S Node  
./tezos-node run --rpc-addr 127.0.0.1`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S Baker  
export TEZOS_LOG='* -> debug'  
./tezos-baker-006-PsCARTHA run with local node ~/.tezos-node baker`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S Endorser  
export TEZOS_LOG='* -> debug'  
./tezos-endorser-006-PsCARTHA run baker`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S Accuser  
export TEZOS_LOG='* -> debug'  
./tezos-accuser-006-PsCARTHA run`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S Ping  
ping -D -O -i 7 X.X.X.X`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

#### Run on Combination Machine \(Node/Baker + Ledger attached on single machine\)

**Run First Time Only to Add Ledger Support and Import Baker’s Key**

First add udev rules then import ledger signer info.

`wget -q -O - https://raw.githubusercontent.com/LedgerHQ/udev-rules/709581c85db97bf6ea12e472aa4e350bf0eabfb7/add_udev_rules.sh | sudo bash  
udevadm trigger  
udevadm control --reload-rules  
./tezos-client list connected ledgers    < — (replace ledger://<4-words-here> commands below with YOUR unique ledger path)  
./tezos-signer import secret key baker "ledger://<4-words-here>/bip25519/0h/0h"    < — (run only first time; to have it available just in case for the other use case with 2 computers)  
./tezos-client import secret key baker "ledger://<4-words-here>/bip25519/0h/0h"    < — (run only first time)`

**Run Every Time: Start Ledger/Node/Baker/Endorser/Accuser Processes**

`screen -S Node  
./tezos-node run --rpc-addr 127.0.0.1`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`./tezos-client setup ledger to bake for baker --main-hwm 1061796    < — (change to latest tezos block` [`www.tzstats.com`](http://www.tzstats.com)`)   
screen -S Baker  
export TEZOS_LOG='* -> debug'  
./tezos-baker-006-PsCARTHA run with local node ~/.tezos-node baker`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S Endorser  
export TEZOS_LOG='* -> debug'  
./tezos-endorser-006-PsCARTHA run baker`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S Accuser  
export TEZOS_LOG='* -> debug'  
./tezos-accuser-006-PsCARTHA run`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

## Updating Tezos Nodes \(for manually compiled\)

{% hint style="info" %}
Make sure all tezos processes are stopped before updating your software. Double check that you don't have any baking of endorsing operations coming up before attemtping to make any changes to your node.
{% endhint %}

First you need to navigate where the Tezos binaries are compiled, then you update the latest source code from the official Tezos source code repository and compile the new version.

`cd ~/tezos  
git fetch && git checkout latest-release && git pull  
make build-deps && eval $(opam env)  
make`

After updating your software you can launch the old binaries and the new binaries to run at the same time.  Once the new protocol is activated or once you bake/endorse your last block for the old protocol, you can discard its binaries.

### Start Node

`screen -S Node  
./tezos-node run --rpc-addr 127.0.0.1`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

Make sure the Ledger Nano S is attached and has the Baking app open. Confirm the command below by hand on your Ledger Nano S.

`./tezos-client setup ledger to bake for baker --main-hwm 1061796    < — (change to latest tezos block` [`www.tzstats.com`](http://www.tzstats.com)`)` 

### Start Old Binaries

`screen -S Baker  
export TEZOS_LOG='* -> debug'  
./tezos-baker-006-PsCARTHA run with local node ~/.tezos-node baker`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S Endorser  
export TEZOS_LOG='* -> debug'  
./tezos-endorser-006-PsCARTHA run baker`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S Accuser  
export TEZOS_LOG='* -> debug'  
./tezos-accuser-006-PsCARTHA run`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

### Start New Binaries

`screen -S BakerNEW  
export TEZOS_LOG='* -> debug'  
./tezos-baker-007-PsDELPH1 run with local node ~/.tezos-node baker`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S EndorserNEW  
export TEZOS_LOG='* -> debug'  
./tezos-endorser-007-PsDELPH1 run baker`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S AccuserNEW  
export TEZOS_LOG='* -> debug'  
./tezos-accuser-007-PsDELPH1 run`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

