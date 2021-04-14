---
description: >-
  The best way to inject operations into the Tezos blockchain and to fetch RPC
  information is from your own node. You get the information faster and you
  trust fewer people.
---

# Setup Tezos Node with RPC

## Prepare to install Tezos node

This guide starts with the latest Ubuntu 20.10 server version but it should work on all Debian based distributions and older \(not ancient\) versions.

### Add non-root user \(for security\)

{% hint style="info" %}
Never run things as a root user. It's not worth the security compromises.

You can skip this step if you're already logging in with a user other than "root"
{% endhint %}

`adduser YOUR_username_here  
usermod -aG sudo YOUR_username_here`

### Install operating system prerequisites

`sudo apt install -y curl xz-utils jq screen build-essential git m4 unzip rsync curl bubblewrap libev-dev libgmp-dev pkg-config libhidapi-dev jbuilder software-properties-common opam autoconf libffi-dev  
sudo apt update`

### Install Tezos prerequisites \[run as regular user\]

#### Install Rust

```text
cd /tmp
wget https://sh.rustup.rs/rustup-init.sh
chmod +x rustup-init.sh
./rustup-init.sh --profile minimal --default-toolchain 1.44.0 -y
```

### Install Zcash Parameters

```text
cd /tmp
https://raw.githubusercontent.com/zcash/zcash/master/zcutil/fetch-params.sh
chmod +x fetch-params.sh
./fetch-params.sh
```

### Download Tezos blockchain snapshot 

{% hint style="info" %}
You can can work on next step while this one runs
{% endhint %}

Use one of the sources below to obtain a full or rolling \(both work for baking\) snapshot of the Tezos blockchain. A rolling snapshot will get you started the fastest and is completely fine to use for most purposes like farming.

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

## Start Tezos node

`screen -S TezosNode  
./tezos-node run --rpc-addr 127.0.0.1`

{% hint style="info" %}
CTRL+A then H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

This starts your node in a "screen" session, which keeps running in the background, as long as you properly exit it.

Once your node is up and fully synchronized, you are ready to inject operations into the blockchain using ./tezos-client or your own RPC running on port 8732 via the Temple wallet.

> 🙏 _Donate & help us grow. All proceeds go to more baking capacity._  
>                                                        **tz1S5WxdZR5f9NzsPXhr7L9L1vrEb5spZFur**

