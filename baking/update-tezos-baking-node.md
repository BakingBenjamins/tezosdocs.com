# Update Tezos Baking Node

{% hint style="info" %}
Update your node to be able to run the Ithaca protocol while still running Hangzhou in production. Once the switchover happens, the Ithaca binaries will take over and you can stop the old ones.
{% endhint %}

{% hint style="info" %}
<mark style="color:red;">**TO RUN ITHACA PROTOCOL YOU MUST UPGRADE YOUR LEDGER BAKING APP TO VERSION 2.2.15 AND THEN IMEDDIATELY RESET THE HIGH WATERMARK ON THE LEDGER AS EXPLAINED BELOW!**</mark>
{% endhint %}

## Updating Tezos Nodes

{% hint style="info" %}
Make sure all Tezos processes are stopped before updating your software. Double check that you don't have any baking of endorsing operations coming up before attempting to make any changes to your node.
{% endhint %}

First you need to navigate where the Tezos binaries are compiled, then you update the latest source code from the official Tezos source code repository and compile the new version.

```
cd ~/tezos
git fetch && git checkout latest-release && git pull
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

Go to [https://tzstats.com/](https://tzstats.com) or [https://tzkt.io](https://tzkt.io) and observe the block heigh, which should be a 7 digit number starting with 2 (example 2220810)

{% hint style="info" %}
<mark style="color:red;">**Replace the 0 below with the current block number.**</mark>  This is important in cases where you've used the Ledger Nano S address to bake in the past.  Since in this case we're resetting a known used Ledger Nano S, we must update the 0 to the latest block to eliminate any possibility of double baking, however remote and minuscule.
{% endhint %}

```
./tezos-client setup ledger to bake for baker --main-hwm 0
```

### Start New Binaries (Ithaca #012)

{% hint style="info" %}
In Ithaca, there is no longer a separate "endorsing" process. With the Ithaca update, the former has been made part of the "baking" process.
{% endhint %}

```
screen -S TezosBaker012
export TEZOS_LOG='* -> debug'
./tezos-baker-012-Psithaca run with local node ~/.tezos-node baker
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S TezosAccuser012
export TEZOS_LOG='* -> debug'
./tezos-accuser-012-Psithaca run
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}



### Start Old Binaries (Hangzhou2 #011)

```
screen -S TezosBaker011
export TEZOS_LOG='* -> debug'
./tezos-baker-011-PtHangz2 run with local node ~/.tezos-node baker
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S TezosEndorser011
export TEZOS_LOG='* -> debug'
./tezos-endorser-011-PtHangz2 run baker
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S TezosAccuser011
export TEZOS_LOG='* -> debug'
./tezos-accuser-011-PtHangz2 run
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}





> 🙏 _Donate & help us grow. All proceeds go to more baking capacity._\
> &#x20;                                                      **tz1S5WxdZR5f9NzsPXhr7L9L1vrEb5spZFur**
