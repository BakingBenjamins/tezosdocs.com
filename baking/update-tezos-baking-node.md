# Update Tezos Baking Node

{% hint style="info" %}
Update your node to be able to run the Granada proposal while still running Florence in production. Once the switchover happens, the Granada binaries will take over and you can stop the old ones.
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

### Start Old Binaries (Florence #009)

```
screen -S TezosBaker
export TEZOS_LOG='* -> debug'
./tezos-baker-009-PsFLoren run with local node ~/.tezos-node baker
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S TezosEndorser009
export TEZOS_LOG='* -> debug'
./tezos-endorser-009-PsFLoren run baker
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S TezosAccuser009
export TEZOS_LOG='* -> debug'
./tezos-accuser-009-PsFLoren run
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

### Start New Binaries (Granada #010)

```
screen -S TezosBaker010
export TEZOS_LOG='* -> debug'
./tezos-baker-010-PtGRANAD run with local node ~/.tezos-node baker
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S TezosEndorser010
export TEZOS_LOG='* -> debug'
./tezos-endorser-010-PtGRANAD run baker
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}

```
screen -S TezosAccuser010
export TEZOS_LOG='* -> debug'
./tezos-accuser-010-PtGRANAD run
```

{% hint style="info" %}
CTRL+A then Shift+H to log/record session\
CTRL+A then d to disconnect from Screen session
{% endhint %}





> 🙏 _Donate & help us grow. All proceeds go to more baking capacity._\
> &#x20;                                                      **tz1S5WxdZR5f9NzsPXhr7L9L1vrEb5spZFur**
