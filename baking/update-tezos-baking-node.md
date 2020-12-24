# Update Tezos Baking Node



## Updating Tezos Nodes

{% hint style="info" %}
Make sure all tezos processes are stopped before updating your software. Double check that you don't have any baking of endorsing operations coming up before attempting to make any changes to your node.
{% endhint %}

First you need to navigate where the Tezos binaries are compiled, then you update the latest source code from the official Tezos source code repository and compile the new version.

`cd ~/tezos  
git fetch  
git checkout latest-release  
git pull  
eval $(opam env)  
make build-deps  
make`

After updating your software you can launch the old binaries and the new binaries to run at the same time.  Once the new protocol is activated or once you bake/endorse your last block for the old protocol, you can discard its binaries.

### Start Universal Node

{% hint style="info" %}
While you need to run 2 copies of the baking, endorsing and accusing binaries, you only have to run one copy of the node binary
{% endhint %}

`screen -S TezosNode  
./tezos-node run --rpc-addr 127.0.0.1`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

Make sure the Ledger Nano S is attached and has the Baking app open. Confirm the command below by hand on your Ledger Nano S.

`./tezos-client setup ledger to bake for baker --main-hwm 1061796  
# (change to latest tezos block)` 

### Start Old Binaries \(Delphi \#007\)

`screen -S TezosBakerDelphi  
export TEZOS_LOG='* -> debug'  
./tezos-baker-007-PsDELPH1 run with local node ~/.tezos-node baker`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S TezosEndorserDelphi  
export TEZOS_LOG='* -> debug'  
./tezos-endorser-007-PsDELPH1 run baker`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S TezosAccuserDelphi  
export TEZOS_LOG='* -> debug'  
./tezos-accuser-007-PsDELPH1 run`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

### Start New Binaries \(Edo \#008\)

`screen -S TezosBakerEdo  
export TEZOS_LOG='* -> debug'  
./tezos-baker-008-PtEdoTez run with local node ~/.tezos-node baker`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S TezosEndorserEdo  
export TEZOS_LOG='* -> debug'  
./tezos-endorser-008-PtEdoTez run baker`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}

`screen -S TezosAccuserEdo  
export TEZOS_LOG='* -> debug'  
./tezos-accuser-008-PtEdoTez run`

{% hint style="info" %}
CTRL+A then Shift+H to log/record session  
CTRL+A then d to disconnect from Screen session
{% endhint %}





> 🙏 _Donate & help us grow. All proceeds go to more baking capacity._  
>                                                        **tz1S5WxdZR5f9NzsPXhr7L9L1vrEb5spZFur**

