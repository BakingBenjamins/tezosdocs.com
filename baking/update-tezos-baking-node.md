---
description: Updating from Lima to Mumbai
---

# Update Tezos Baking Node

Update your node to be able to run the Mumbai protocol while still running Lima in production. Once the switchover happens, the Ithaca binaries will take over and you can stop the old ones.

## Updating Tezos Nodes

{% hint style="info" %}
Make sure all Tezos processes are stopped before updating your software. Double check that you don't have any baking of endorsing operations coming up before attempting to make any changes to your node.
{% endhint %}

First you need to navigate where the Tezos binaries are compiled, then you update the latest source code from the official Tezos source code repository and compile the new version.

```
cd ~/tezos
git pull
git checkout latest-release
rm -rf _opam _build
make build-deps
eval $(opam env)
make
```

After updating your software you can launch the old binaries and the new binaries to run at the same time.  Once the new protocol is activated or once you bake/endorse your last block for the old protocol, you can discard its binaries.

### Start Universal Node

{% hint style="info" %}
While you need to run 2 copies of the baking, endorsing and accusing binaries, you only have to run one copy of the node binary
{% endhint %}

```
screen -S TezosNode
./tezos-node run --rpc-addr 127.0.0.1
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

Make sure your Ledger Nano S has version 2.2.15 of the baking app. Once updated, the ledger needs to run the following command immediately or baking will not work. The reason you must update the high watermark is because the way it functions has changed in the Tenderbake consensus.

### Start New Binaries (Mumbai)

{% hint style="info" %}
In Ithaca, there is no longer a separate "endorsing" process. With the Ithaca update, the former has been made part of the "baking" process.
{% endhint %}

{% hint style="info" %}
**--liquidity-baking-toggle-vote is now mandatory for the baker daemon. Your options are:**\
**pass, yes, no (lower case).**&#x20;

**This determines how you vote for Liquidity Baking. Read more here:** [https://news.tezoscommons.org/liquidity-baking-on-tezos-what-to-expect-151be29aa0ed](https://news.tezoscommons.org/liquidity-baking-on-tezos-what-to-expect-151be29aa0ed)
{% endhint %}

```
screen -S TezosBakerMumbai
export TEZOS_LOG='* -> debug'
./octez-baker-PtMumbai run with local node ~/.tezos-node baker --liquidity-baking-toggle-vote pass
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S TezosAccuserMumbai
export TEZOS_LOG='* -> debug'
./octez-accuser-PtMumbai run
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}



### Start Old Binaries (Lima)

{% hint style="info" %}
In Ithaca, there is no longer a separate "endorsing" process. With the Ithaca update, the former has been made part of the "baking" process.
{% endhint %}

```
screen -S TezosBakerLima
export TEZOS_LOG='* -> debug'
./octez-baker-PtLimaPt run with local node ~/.tezos-node baker
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S TezosAccuserLima
export TEZOS_LOG='* -> debug'
./octez-accuser-PtLimaPt run
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}







> 🙏 _Donate & help us grow. All proceeds go to more baking capacity._\
> &#x20;                                                      **tz1S5WxdZR5f9NzsPXhr7L9L1vrEb5spZFur**
