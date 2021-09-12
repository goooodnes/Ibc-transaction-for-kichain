# Ibc-transaction kichain-t-4 umee-betanet-1
# RELAYER located 19.13.183.88888, where UMEE node located 19.13.183.88888 (address number was changed of course)
# You can use global rpc server for kichain https://rpc-challenge.blockchain.ki:443
# check "sync_info" of your UMEE node: 
# "catching_up": true # then wait for false status, or relayer cant esteblish channel till your node unsync
# "catching_up": false # then go

root@vmi664097:~# curl localhost:26657/statusr
{
      "catching_up": false
}

         
Download and install the Relayer v.1.0.0


> git clone https://github.com/cosmos/relayer.git

> cd relayer

> make install

> cd

> rly version

output:

version: 1.0.0-rc1–152-g112205b

 
 ##                                                  Initializing the repeater

> rly config init

create a folder with network configuration and go to it:

> mkdir rly_config

> cd rly_config

##                               create a settings file for both networks:

> nano kichain-t-4.json
 # GLOBAL RPC its simple way
 {  "chain-id": "kichain-t-4",   "rpc-addr": "https://rpc-challenge.blockchain.ki:443",    "account-prefix": "tki",   "gas-adjustment": 1.5,   "gas-prices": "0.025utki",   "trusting-period": "48h" } 
 
 # remote NODE  for those who dont like simle ways, replace http://62.71.999999.22:26657 with address of you kichain node
 {  "chain-id": "kichain-t-4",   "rpc-addr": "http://62.71.999999.22:26657",    "account-prefix": "tki",   "gas-adjustment": 1.5,   "gas-prices": "0.025utki",   "trusting-period": "48h" }  
 # DO in YOUR remote node KICHAIN change http://62.71.999999.22:26657 with address of you kichain node
root@vmi648853:~# find -name config.toml
./kid-backup/config/config.toml
./kid-kichain-t-3-backup/config/config.toml
./.kid/config/config.toml
./kichain/kid/config/config.toml
root@vmi648853:~# nano ./kichain/kid/config/config.toml

# find rpc 127.0.0.1:26657 replace on 0.0.0.0:26657
#######################################################
###       RPC Server Configuration Options          ###
#######################################################
[rpc]

# TCP or UNIX socket address for the RPC server to listen on
laddr = "tcp://127.0.0.1:26657" # replace
laddr = "tcp://0.0.0.0:26657"   # for this to have remote access

# on the machine where RELAYER located 19.13.183.88888, where UMEE node located 19.13.183.88888 (address number was changed of course)
root@vmi664097:~/rly_config# 
nano umee-betanet-1.json
{ "chain-id": "umee-betanet-1", "rpc-addr": "http://127.0.0.1:26657", "account-prefix": "umee", "gas-adjustment": 1.5, "gas-prices": "0.025uumee", "trusting-period": "48h" }
root@vmi664097:~/rly_config# ls
kichain-t-4.json  umee-betanet-1.json



#### We parse these settings into the config of the relay:

root@vmi664097:~/rly_config# rly chains add -f umee-betanet-1.json && rly chains add -f kichain-t-4.json && cd
 
#### restore WALLETS it with the command:

> rly keys restore umee-betanet-1 name of your wallet "YOUR MNEMONIC PHRAZE HERE"
> rly keys restore kichain-t-4 name of your wallet "YOUR MNEMONIC PHRAZE HERE"

#### Add the newly created keys to the config of the relay:
> rly chains edit umee-betanet-1 key name of your wallet
>
> rly chains edit kichain-t-4 key name of your wallet

 
#### Wallets must be funded on both networks. check the availability of coins with the command:
> rly q balance umee-betanet-1
>
> rly q balance kichain-t-4

#### If there are coins on the balance and the relay shows them, then we continue, We initialize the light client in both networks with the command:

> rly light init umee-betanet-1 -f
>
> rly light init kichain-t-4 -f

#### We try to generate a channel between the networks with the command:

> rly paths generate kichain-t-4 umee-betanet-1 ku 

> Generated path(transfer), run 'rly paths show ku --yaml' to see details
path:
  src:
    chain-id: kichain-t-4
    client-id: 07-tendermint-267
    connection-id: connection-281
    channel-id: channel-228
    port-id: transfer
    order: UNORDERED
    version: ics20-1
  dst:
    chain-id: umee-betanet-1
    client-id: 07-tendermint-9
    connection-id: connection-46
    channel-id: channel-41
    port-id: transfer
    order: UNORDERED
    version: ics20-1
  strategy:
    type: naive
status:
  chains: true
  clients: true
  connection: true
  channel: true


 #### Trying to open a channel for relaying:

> rly tx link ku  --debug

# you can replace the name of channel "ku" on something else like "ki_umee", or whatever name of channel

> root@vmi664097:~# rly tx link ku --debug

I[2021-09-12|11:22:50.045] ★ Channel created: [kichain-t-4]chan{channel-228}port{transfer} -> [umee-betanet-1]chan{channel-41}port{transfer}


#### if it so happens that the channel does not open, then go to the config with the command:

> nano /root/.relayer/config/config.yaml

#### paste next, dont forget replace kovalenkoandrii_umee_keys_wallet and kovalenkoandriiwallet on your keys:

global:
  api-listen-addr: :5183
  timeout: 10s
  light-cache-size: 20
chains:
- key: kovalenkoandrii_umee_keys_wallet
  chain-id: umee-betanet-1
  rpc-addr: http://127.0.0.1:26657
  account-prefix: umee
  gas-adjustment: 1.5
  gas-prices: 0.025uumee
  trusting-period: 48h
- key: kovalenkoandriiwallet
  chain-id: kichain-t-4
  rpc-addr: https://rpc-challenge.blockchain.ki:443
  account-prefix: tki
  gas-adjustment: 1.5
  gas-prices: 0.025utki
  trusting-period: 48h
paths:
  ku:
    src:
      chain-id: kichain-t-4
      client-id: 07-tendermint-267
      connection-id: connection-281
      channel-id: channel-228
      port-id: transfer
      order: UNORDERED
      version: ics20-1
    dst:
      chain-id: umee-betanet-1
      client-id: 07-tendermint-9
      connection-id: connection-46
      channel-id: channel-41
      port-id: transfer
      order: UNORDERED
      version: ics20-1
    strategy:
      type: naive


> root@vmi664097:~# rly paths list -d
 0: ku                   -> chns(✔) clnts(✔) conn(✔) chan(✔) (kichain-t-4:transfer<>umee-betanet-1:transfer)
root@vmi664097:~# rly chains list
 0: umee-betanet-1       -> key(✔) bal(✔) light(✔) path(✔)
 1: kichain-t-4          -> key(✔) bal(✔) light(✔) path(✔)

# below from kichain-t-4 to umee-betanet-1 1000utki  transfer example, dont forget to replace destination wallet umee1450qcyclf295v00w6rrkajxu2djapfjyuj90x5 with your one)

root@vmi664097:~# rly tx transfer kichain-t-4 umee-betanet-1 1000utki umee1450qcyclf295v00w6rrkajxu2djapfjyuj90x5 --path ku
I[2021-09-12|11:22:53.372] ✔ [kichain-t-4]@{296114} - msg(0:transfer) hash(7354F2C4615AC841373AAC23E2B020D4003726B2DF55D68679FC723DC656CC84)
# below from umee-betanet-1 to kichain-t-4 2000uume  transfer example, dont forget to replace destination wallet tki1kj6gvafwp0g9ezp6faxpl8k8y968p7trjepcmu with your one)

root@vmi664097:~# rly tx transfer umee-betanet-1 kichain-t-4 2000uumee tki1kj6gvafwp0g9ezp6faxpl8k8y968p7trjepcmu --path ku
I[2021-09-12|11:31:31.026] ✔ [umee-betanet-1]@{282069} - msg(0:transfer) hash(87978763F8349FE099227EE630CAC955BDE4C9424A41E37795851E61C0E6C1C2)
root@vmi664097:~# rly tx transfer umee-betanet-1 kichain-t-4 4000uumee tki1kj6gvafwp0g9ezp6faxpl8k8y968p7trjepcmu --path ku
I[2021-09-12|11:40:27.099] ✔ [umee-betanet-1]@{282154} - msg(0:transfer) hash(CB2590C53F4058BF77969519814D06972D6410A457E70EE2BFB7DF499EAC0B01)
root@vmi664097:~# rly tx transfer kichain-t-4 umee-betanet-1 3000utki umee1450qcyclf295v00w6rrkajxu2djapfjyuj90x5 --path ku
I[2021-09-12|11:40:49.363] ✔ [kichain-t-4]@{296283} - msg(0:transfer) hash(701DA068BEA54510730BF0E8D4C3DDF7A9E0B964D58F2D655514BEE69CBC1FB5)

# to see transaction on the explorer, dont forget to replace hash 87978763F8349FE099227EE630CAC955BDE4C9424A41E37795851E61C0E6C1C2 on yours)
https://testnet.mintscan.io/umee/txs/87978763F8349FE099227EE630CAC955BDE4C9424A41E37795851E61C0E6C1C2

https://testnet.mintscan.io/umee/txs/CB2590C53F4058BF77969519814D06972D6410A457E70EE2BFB7DF499EAC0B01

https://ki.thecodes.dev/tx/7354F2C4615AC841373AAC23E2B020D4003726B2DF55D68679FC723DC656CC84

https://ki.thecodes.dev/tx/701DA068BEA54510730BF0E8D4C3DDF7A9E0B964D58F2D655514BEE69CBC1FB5


#### if you have forgotten wallet addresses. this happens sometimes :-) then you can see the command

> rly keys list umee-betanet-1
>
> rly keys list kichaint-t-4


# logs ouput with changes

root@vmi664097:~# osmosisd query bank balances osmo1k06y45zh9xm2yt07hvgzsjesth4sh3a9ehmh69 --node http://5.1.1.:36657
balances: []
pagination:
  next_key: null
  total: "0"
root@vmi664097:~# curl localhost:26657/status
{
  "jsonrpc": "2.0",
  "id": -1,
  "result": {
    "node_info": {
      "protocol_version": {
        "p2p": "8",
        "block": "11",
        "app": "0"
      },
      "id": "f94ddb00725e505c43b53051260ceb5490b0b0a1",
      "listen_addr": "tcp://0.0.0.0:26656",
      "network": "umee-betanet-1",
      "version": "v0.34.12",
      "channels": "40202122233038606100",
      "moniker": "kovalenkoandrii_umee_node",
      "other": {
        "tx_index": "on",
        "rpc_address": "tcp://127.0.0.1:26657"
      }
    },
    "sync_info": {
      "latest_block_hash": "64CEFD079A0786DF2B0B414325123E66BA3DB9EC3CE54C46E298AED6B6DFBBB1",
      "latest_app_hash": "8D3D042B087F910055B5033EB992168749E5D6464B075DD6AB28C3740ADF99FE",
      "latest_block_height": "281634",
      "latest_block_time": "2021-09-12T08:48:24.036183287Z",
      "earliest_block_hash": "AF53DD4EBB959B6A4FD716C9225E74EF95F87FBFB204C157DD8C3933E9BEC017",
      "earliest_app_hash": "E3B0C44298FC1C149AFBF4C8996FB92427AE41E4649B934CA495991B7852B855",
      "earliest_block_height": "1",
      "earliest_block_time": "2021-08-24T16:12:42.679687705Z",
      "catching_up": false
    },
    "validator_info": {
      "address": "CA17ECA54AB1A728FA0611B38BDDF7C71B154F2F",
      "pub_key": {
        "type": "tendermint/PubKeyEd25519",
        "value": "8lJMp2YzSdu5Op9aW3I0sJDQKsWuXMMhrO/iwYbB2go="
      },
      "voting_power": "0"
    }
  }
}root@vmi664097:~# cd rly_config/
root@vmi664097:~/rly_config# ls
cygnusx-osmo-1_config.json  flixnet-2.json  groot-011.json  kichain-t-4.json  testnet-croeseid-4.json  umee-betanet-1.json
root@vmi664097:~/rly_config# nano umee-betanet-1.json
root@vmi664097:~/rly_config# rly chains add -f umee-betanet-1.json
Error: chain with ID umee-betanet-1.json already exists in config
root@vmi664097:~/rly_config# cd
root@vmi664097:~# rly keys restore umee-betanet-1 kovalenkoandrii_umee_keys_wallet "........... mobile economy motion mix tooth mixed sustain cliff"
Error: chain with ID umee-betanet-1 is not configured
root@vmi664097:~# rm -rf .relayer
root@vmi664097:~# rly config init
root@vmi664097:~# cd rly_config/
root@vmi664097:~/rly_config# rly chains add -f umee-betanet-1.json
root@vmi664097:~/rly_config# cd
root@vmi664097:~# rly keys restore umee-betanet-1 kovalenkoandrii_umee_keys_wallet "........... mobile economy motion mix tooth mixed sustain cliff"
Error: chain with ID umee-betanet-1 is not configured
root@vmi664097:~# rly keys add umee-betanet-1 umeekey
Error: chain with ID umee-betanet-1 is not configured
root@vmi664097:~# cd rly_config/
root@vmi664097:~/rly_config# nano umee-betanet-1.json
root@vmi664097:~/rly_config# cd
root@vmi664097:~# rly keys restore umee-betanet-1 kovalenkoandrii_umee_keys_wallet "........... economy motion mix tooth mixed sustain cliff"
Error: chain with ID umee-betanet-1 is not configured
root@vmi664097:~# rly chains edit umee-betanet-1.json chain-id umee-betanet-1
Error: chain with ID umee-betanet-1 already exists in config
root@vmi664097:~# rm -rf .relayer
root@vmi664097:~# rly config init
root@vmi664097:~# cd rly_config/
root@vmi664097:~/rly_config# rly chains add -f umee-betanet-1.json
root@vmi664097:~/rly_config# cd
root@vmi664097:~# rly keys restore umee-betanet-1 kovalenkoandrii_umee_keys_wallet "..................harvest mobile economy motion mix tooth mixed sustain cliff"
umee1450qcyclf295v00w6rrkajxu2djapfjyuj90x5
root@vmi664097:~# rly chains edit umee-betanet-1 key kovalenkoandrii_umee_keys_wallet && rly q balance umee-betanet-1 kovalenkoandrii_umee_keys_wallet
100000000uumee
root@vmi664097:~# cd rly_config/
root@vmi664097:~/rly_config# nano kichain-t-4.json
root@vmi664097:~/rly_config# rly chains add -f kichain-t-4.json
root@vmi664097:~/rly_config# cd
root@vmi664097:~# rly keys restore kichain-t-4 kovalenkoandriiwallet "..........  leader green panda piece zoo emerge coil resemble"
tki1kj6gvafwp0g9ezp6faxpl8k8y968p7trjepcmu
root@vmi664097:~# rly chains edit kichain-t-4 key kovalenkoandriiwallet && rly q balance kichain-t-4 kovalenkoandriiwallet && rly light init kichain-t-4 -f
931604565utki
successfully created light client for kichain-t-4 by trusting endpoint http://2.1.188888883.220:26657...
root@vmi664097:~# rly light init umee-betanet-1 -f && rly paths gen kichain-t-4 umee-betanet-1 ku
successfully created light client for umee-betanet-1 by trusting endpoint http://127.0.0.1:26657...
Generated path(ku), run 'rly paths show ku --yaml' to see details
root@vmi664097:~# rly paths show ku --yaml
path:
  src:
    chain-id: kichain-t-4
    port-id: transfer
    order: UNORDERED
    version: ics20-1
  dst:
    chain-id: umee-betanet-1
    client-id: 07-tendermint-9
    port-id: transfer
    order: UNORDERED
    version: ics20-1
  strategy:
    type: naive
status:
  chains: true
  clients: false
  connection: false
  channel: false

root@vmi664097:~# rly tx link ku --debug
I[2021-09-12|11:12:49.865] - [kichain-t-4] -> creating client on kichain-t-4 for umee-betanet-1 header-height{281884} trust-period(48h0m0s)
I[2021-09-12|11:12:57.914] ✔ [kichain-t-4]@{296020} - msg(0:create_client) hash(E7B98CCDE4D1F780C35C9C7BCD08E475C871CE256EEC95400F848AABC1D397F3)
I[2021-09-12|11:12:57.915] ★ Clients created: client(07-tendermint-267) on chain[kichain-t-4] and client(07-tendermint-9) on chain[umee-betanet-1]
I[2021-09-12|11:12:58.273] - attempting to create new connection ends from chain[kichain-t-4] with chain[umee-betanet-1]
I[2021-09-12|11:13:04.152] ✔ [kichain-t-4]@{296021} - msg(0:update_client,1:connection_open_init) hash(CB669876BE519087E50DA6AA8283EA39E2A218DDFF27C96176F762F35D06A31B)
I[2021-09-12|11:13:08.308] - chain[umee-betanet-1] trying to open connection end on chain[kichain-t-4]
rly tx transfer kichain-t-4 umee-betanet-1 1000utki umee1450qcyclf295v00w6rrkajxu2djapfjyuj90x5 --path ku
I[2021-09-12|11:21:26.846] cannot submit an empty proof init: invalid proof [cosmos/cosmos-sdk@v0.42.4/x/ibc/core/03-connection/types/msgs.go:161]
I[2021-09-12|11:21:26.849] retrying transaction...
I[2021-09-12|11:21:32.282] - chain[umee-betanet-1] trying to open connection end on chain[kichain-t-4]
I[2021-09-12|11:21:38.959] ✔ [umee-betanet-1]@{281974} - msg(0:update_client,1:connection_open_try) hash(65CE742CC594DE1FCEA4222B195F09FCE1860739E09F865C806B4667E8DEE347)
I[2021-09-12|11:21:48.290] - [kichain-t-4]@{{4 296103}}conn(connection-281)-{STATE_INIT} : [umee-betanet-1]@{{1 281975}}conn(connection-46)-{STATE_TRYOPEN}
I[2021-09-12|11:21:52.179] ✔ [kichain-t-4]@{296104} - msg(0:update_client,1:connection_open_ack) hash(6DB31A422CC02A660A27A4C7447A16B199A0A0340283C025108CF7E13835ABD0)
I[2021-09-12|11:21:58.283] - [umee-betanet-1]@{{1 281977}}conn(connection-46)-{STATE_TRYOPEN} : [kichain-t-4]@{{4 296105}}conn(connection-281)-{STATE_OPEN}
I[2021-09-12|11:22:02.643] ✔ [umee-betanet-1]@{281978} - msg(0:update_client,1:connection_open_confirm) hash(4F633D6EF7766D3FF19CD3381E669F77EAF42519EB70BF81334342625EF166AE)
I[2021-09-12|11:22:02.677] - [kichain-t-4]@{{4 296106}}conn(connection-281)-{STATE_OPEN} : [umee-betanet-1]@{{1 281978}}conn(connection-46)-{STATE_TRYOPEN}
I[2021-09-12|11:22:02.677] ★ Connection created: [kichain-t-4]client{07-tendermint-267}conn{connection-281} -> [umee-betanet-1]client{07-tendermint-9}conn{connection-46}
I[2021-09-12|11:22:02.903] - attempting to create new channel ends from chain[kichain-t-4] with chain[umee-betanet-1]
I[2021-09-12|11:22:10.875] ✔ [kichain-t-4]@{296107} - msg(0:update_client,1:channel_open_init) hash(67A7B483C044DDBB1DEFA8714CDA4AA4B18C30642BA2D3D1BE27E8A4066C0B6E)
I[2021-09-12|11:22:13.059] - chain[umee-betanet-1] trying to open channel end on chain[kichain-t-4]
I[2021-09-12|11:22:13.597] rpc error: code = InvalidArgument desc = cannot submit an empty proof init: invalid proof: invalid request
I[2021-09-12|11:22:13.597] retrying transaction...
I[2021-09-12|11:22:22.989] - chain[umee-betanet-1] trying to open channel end on chain[kichain-t-4]
I[2021-09-12|11:22:26.636] ✔ [umee-betanet-1]@{281982} - msg(0:update_client,1:channel_open_try) hash(ED0417901EE489AB5B724AE9839E6E3F8345B2CA0F1355D002B4FF1C8DDA3329)
I[2021-09-12|11:22:33.057] - [kichain-t-4]@{{4 296110}}chan(channel-228)-{STATE_INIT} : [umee-betanet-1]@{{1 281983}}chan(channel-41)-{STATE_TRYOPEN}
I[2021-09-12|11:22:35.320] ✔ [kichain-t-4]@{296111} - msg(0:update_client,1:channel_open_ack) hash(A3DB18F0D92306B33997F65C8660B66A1B38CFB0EB194D351E159C239D9238BC)
I[2021-09-12|11:22:43.068] - [umee-betanet-1]@{{1 281984}}chan(channel-41)-{STATE_TRYOPEN} : [kichain-t-4]@{{4 296112}}chan(channel-228)-{STATE_OPEN}
I[2021-09-12|11:22:49.981] ✔ [umee-betanet-1]@{281986} - msg(0:update_client,1:channel_open_confirm) hash(069485B4BCB4098B81193D3497FECFF1C5A5C474890D96E55C4D432CD8157FAD)
I[2021-09-12|11:22:50.045] - [kichain-t-4]@{{4 296113}}chan(channel-228)-{STATE_OPEN} : [umee-betanet-1]@{{1 281985}}chan(channel-41)-{STATE_TRYOPEN}
I[2021-09-12|11:22:50.045] ★ Channel created: [kichain-t-4]chan{channel-228}port{transfer} -> [umee-betanet-1]chan{channel-41}port{transfer}
root@vmi664097:~# rly tx transfer kichain-t-4 umee-betanet-1 1000utki umee1450qcyclf295v00w6rrkajxu2djapfjyuj90x5 --path ku
I[2021-09-12|11:22:53.372] ✔ [kichain-t-4]@{296114} - msg(0:transfer) hash(7354F2C4615AC841373AAC23E2B020D4003726B2DF55D68679FC723DC656CC84)
root@vmi664097:~# rly tx transfer umee-betanet-1 kichain-t-4 2000uumee tki1kj6gvafwp0g9ezp6faxpl8k8y968p7trjepcmu --path ku
I[2021-09-12|11:31:31.026] ✔ [umee-betanet-1]@{282069} - msg(0:transfer) hash(87978763F8349FE099227EE630CAC955BDE4C9424A41E37795851E61C0E6C1C2)
root@vmi664097:~# rly tx transfer umee-betanet-1 kichain-t-4 4000uumee tki1kj6gvafwp0g9ezp6faxpl8k8y968p7trjepcmu --path ku
I[2021-09-12|11:40:27.099] ✔ [umee-betanet-1]@{282154} - msg(0:transfer) hash(CB2590C53F4058BF77969519814D06972D6410A457E70EE2BFB7DF499EAC0B01)
root@vmi664097:~# rly tx transfer kichain-t-4 umee-betanet-1 3000utki umee1450qcyclf295v00w6rrkajxu2djapfjyuj90x5 --path ku
I[2021-09-12|11:40:49.363] ✔ [kichain-t-4]@{296283} - msg(0:transfer) hash(701DA068BEA54510730BF0E8D4C3DDF7A9E0B964D58F2D655514BEE69CBC1FB5)
root@vmi664097:~# rly paths show ku --yaml
path:
  src:
    chain-id: kichain-t-4
    client-id: 07-tendermint-267
    connection-id: connection-281
    channel-id: channel-228
    port-id: transfer
    order: UNORDERED
    version: ics20-1
  dst:
    chain-id: umee-betanet-1
    client-id: 07-tendermint-9
    connection-id: connection-46
    channel-id: channel-41
    port-id: transfer
    order: UNORDERED
    version: ics20-1
  strategy:
    type: naive
status:
  chains: true
  clients: true
  connection: true
  channel: true

root@vmi664097:~# nano /root/.relayer/config/config.yaml
root@vmi664097:~# rly paths list -d
 0: ku                   -> chns(✔) clnts(✔) conn(✔) chan(✔) (kichain-t-4:transfer<>umee-betanet-1:transfer)
root@vmi664097:~# rly chains list
 0: umee-betanet-1       -> key(✔) bal(✔) light(✔) path(✔)
 1: kichain-t-4          -> key(✔) bal(✔) light(✔) path(✔)

