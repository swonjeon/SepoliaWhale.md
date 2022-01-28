# Testnets after merge

Early 2022, Ethereum is still PoW and motoring towards merge / PoS. Surviving testnets will likely be Sepolia - currently PoW - and Goerli, as the legacy testnet. Plus whatever ACD (AllCoreDevs) comes up with in addition. Ropsten is dead, ditto Rinkeby, and Kovan is Gnosis' concern. Probably. Reading tea leaves, until there's an official announcement.

I want to "get ready" for some Sepolia testing pre- and post-merge. As it's PoW, I can actually go mine it. Here's how.

## Components

- Geth on Sepolia, Linux. This could absolutely be Windows, I just happen to have Linux tooling I like
- ethminer, Windows, with an NVidia or AMD GPU. Mostly because the only GPU I have that can mine is in my Windows box

## Set up Geth

I'm using [eth-docker](https://eth-docker.net) for this. 

Get prereqs: `sudo apt update && sudo apt install -y docker-compose`

Clone eth-docker: `git clone https://github.com/eth-educators/eth-docker.git sepolia && cd sepolia`

Configure eth-docker for this: `nano .env` and then set

- `COMPOSE_FILE=geth.yml:ec-shared.yml`
- `EC_NETWORK=sepolia`

Save and close

Get the ETH address you want the Sepolia mining rewards to be sent to. Anything created in Metamask will do.

A few changes for Geth so it can mine. `nano geth.yml` and change the `entrypoint`. Enable the `miner` API, and append to it:

```
    entrypoint:
      - geth
      .... existing entries, leave those be
      - --http.api
      - web3,eth,net,miner
      .... existing entries, leave those be
      - --pprof.addr
      - 0.0.0.0
      - --mine
      - --miner.etherbase
      - "0xMYETHADDRESS"
```

You need the quotes around the address, and `0xMYETHADDRESS` is going to be your actual address.

Start it: `./ethd up`, and look at the logs with `./ethd logs -f execution`. It should sync the chain in minutes and then be ready for mining.

## Firewalling

If your Linux boxen is on the Internet, you'll want to firewall the RPC port 8545 and WS port 8546, which are both exposed now. Follow
[instructions](https://eth-docker.net/docs/Support/Cloud) to place ufw "in front of" docker, get the public IP your Windows box uses
with something like [whatismyip](https://whatismyip.com) and create a simple policy along these lines:

```
sudo ufw allow OpenSSH
sudo ufw allow proto tcp from MYPUBLICIP to any port 8545
sudo ufw deny 8545
sudo ufw deny 8546
```

And finally `sudo ufw enable`. Adjust the policy to your own needs if you already had ufw running.

## Windows mining

Grab [ethminer](https://github.com/ethereum-mining/ethminer/releases). It's open source and can use getwork from Geth, not just pools
like closed-source miners.

Extract that somewhere, get into a PowerShell, then start it like so: For NVidia, `.\ethminer.exe -U -P getwork://IP-OF-GETH-BOX:8545` or for AMD, `.\ethminer.exe -G -P getwork://IP-OF-GETH-BOX:8545`.

You can get help with `.\ethminer.exe --help`

## Done

In your Geth logs, you should see `🔨 mined potential block` and `🔗 block reached canonical chain` as well as the occasional
`⑂ block became an uncle`, and in ethminer you should see `**Accepted` messages.

Enjoy the feeling of early Ethereum mining. Difficulty on Sepolia is low, you should see rewards come in steadily.
