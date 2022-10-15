---
description: How to compile Tezos from source and run node software in "screen" sessions.
---

# Setup Tezos Baking Node

This guide will show how to manually compile Tezos binaries from source code.  You can find more information on the Tezos source code website: [https://tezos.gitlab.io/introduction/howtoget.html#compiling-with-make](https://tezos.gitlab.io/introduction/howtoget.html#compiling-with-make)

**If you'd like to get to baking faster and simpler, check out** [**https://bakebuddy.xyz** ](https://bakebuddy.xyz)****

## Prepare to install Tezos node

This guide starts with the latest Ubuntu 22.04 TLS server version but it should work on all Debian based distributions and older (not ancient) versions.

### Install operating system prerequisites

```
sudo apt update
sudo apt install -y curl xz-utils jq screen build-essential git m4 unzip rsync curl bubblewrap libev-dev libgmp-dev pkg-config libhidapi-dev jbuilder software-properties-common opam autoconf libffi-dev patch wget g++ zlib1g-dev bc
```

### Install Tezos prerequisites

#### Install Rust

```
cd /tmp
wget https://sh.rustup.rs/rustup-init.sh
chmod +x rustup-init.sh
./rustup-init.sh --profile minimal --default-toolchain 1.60.0 -y
```

### Install Zcash Parameters

```
cd /tmp
wget https://raw.githubusercontent.com/zcash/zcash/master/zcutil/fetch-params.sh
chmod +x fetch-params.sh
./fetch-params.sh
```

### Download Tezos blockchain snapshot&#x20;

{% hint style="info" %}
You can can work on next step while this one runs
{% endhint %}

Use one of the sources below to obtain a full or rolling (both work for baking) snapshot of the Tezos blockchain. A rolling snapshot will get you started the fastest and is completely fine to use for baking nodes.

{% embed url="https://xtz-shots.io/" %}
MIDL
{% endembed %}

{% embed url="https://snaps.teztools.io/" %}
TezTools
{% endembed %}

## Compile Tezos node & import snapshot

### Download Tezos source code, initialize opam and build Tezos binaries

```
cd ~ && git clone https://gitlab.com/tezos/tezos.git && cd tezos && git checkout latest-release
opam init --bare
# (answer yes to questions)
make build-deps 
eval $(opam env)
make
```

### Generate Tezos node identity

```
cd ~/tezos && ./tezos-node identity generate
```

### Import chain snapshot and verify its legitimacy

{% hint style="info" %}
The import of the snapshot will take a no more than an hour on most systems.  If you're importing a full snapshot, the process may take up to a full day if you're running on a slow VM or fanless system
{% endhint %}

```
cd ~/tezos && ./tezos-node snapshot import /path/to/chain.full --block <ENTER_BLOCK_HASH_HERE>
```

Replace the made up path to where your downloaded and uncompressed the Tezos chain snapshot. Navigate to any Tezos blockchain explorer like [https://tzstats.com](https://tzstats.com) and [https://tzkt.io](https://tzkt.io) and look up the block number referenced by the snapshot website. The block will have a hash # associated with it, which you will need to copy into the `<ENTER_BLOCK_HASH_HERE>` portion. This verifies the integrity of the chain and saves you many hours of synchronization and waiting.

## Start Tezos node and baking executables

### **Run First Time Only to Add Ledger Support, Import Baker’s Key and register as baker**

{% hint style="info" %}
During setup have the Ledger Nano S Tezos baker application open for all steps until the last one, for which open the Tezos wallet application.  After this, you will never need to use the Tezos wallet app to bake and you should ALWAYS have the Tezos baker applic
{% endhint %}

First add udev rules then import ledger signer info.

```
wget -q -O - https://raw.githubusercontent.com/LedgerHQ/udev-rules/709581c85db97bf6ea12e472aa4e350bf0eabfb7/add_udev_rules.sh | sudo bash
udevadm trigger
udevadm control --reload-rules
cd ~/tezos

./tezos-client list connected ledgers

./tezos-client import secret key baker "ledger://<4-words-here>/ed25519/0h/0h"
(note: replace ledger://<4-words-here> commands below with YOUR unique ledger path)

./tezos-client register key baker as delegate

./tezos-client setup ledger to bake for baker --main-hwm 0
(note: replace 0 with the current block if you've even baked with this ledger device before)
```

### **Run Every Time: Start Ledger/Node/Baker/Endorser/Accuser Processes**

```
cd ~/tezos
screen -S OctezNode
./tezos-node run --rpc-addr 127.0.0.1
```

{% hint style="info" %}
CTRL+A then H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S OctezBaker014
export TEZOS_LOG='* -> debug'
./tezos-baker-014-PtKathma run with local node ~/.tezos-node baker
```

{% hint style="info" %}
CTRL+A then H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S OctezAccuser014
export TEZOS_LOG='* -> debug'
./tezos-accuser-014-PtKathma run
```

{% hint style="info" %}
CTRL+A then H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

You can monitor your node by running:

```
cd ~/tezos
tail -f screen*
```



> 🙏 _Donate & help us grow. All proceeds go to more baking capacity._\
> &#x20;                                                      **tz1S5WxdZR5f9NzsPXhr7L9L1vrEb5spZFur**
