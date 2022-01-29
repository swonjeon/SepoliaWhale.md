# Testnets after merge

Early 2022, Ethereum is still PoW and motoring towards merge / PoS. [Surviving testnets](https://github.com/ethereum/pm/issues/460) will likely be Sepolia - currently PoW - and Goerli. Plus whatever ACD (AllCoreDevs) comes up with in addition. Ropsten is dead, ditto Rinkeby, and Kovan is Gnosis' concern. Probably. Reading tea leaves, until there's an official announcement.

I want to "get ready" for some Sepolia testing pre- and post-merge. As it's PoW, I can actually go mine myself some SEPPETH. Here's how.

## Components

- Geth on Sepolia, Linux. This could absolutely be Windows, I just happen to have Linux tooling I like
- ethminer or t-rex, Windows, with an NVidia or AMD GPU. Mostly because the only GPU I have that can mine is in my Windows box

## Set up Geth

I'm using [eth-docker](https://eth-docker.net) for this. 

Get prereqs: `sudo apt update && sudo apt install -y docker-compose`

Clone eth-docker: `git clone https://github.com/eth-educators/eth-docker.git sepolia && cd sepolia`

Configure eth-docker for this: `cp default.env .env`, then `nano .env` and set

- `COMPOSE_FILE=geth.yml:ec-shared.yml`
- `EC_NETWORK=sepolia`

Save and close.

Get the ETH address you want the Sepolia mining rewards to be sent to. Anything created in Metamask will do.

A few changes for Geth so it can mine. `nano geth.yml` and change the `entrypoint`. Enable the `miner` API, and append to it:

```
    entrypoint:
      - geth
      - --http.corsdomain=*
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

If your Linux boxen is on the Internet, not in your home network, *and only then*, you'll want to firewall the RPC port 8545 and WS port 8546, which are both exposed now. Follow
[instructions](https://eth-docker.net/docs/Support/Cloud) to place ufw "in front of" docker, get the public IP your Windows box uses
with something like [whatismyip](https://whatismyip.com) and create a simple policy along these lines:

```
sudo ufw allow OpenSSH
sudo ufw allow proto tcp from MYPUBLICIP to any port 8545
sudo ufw deny 8545
sudo ufw deny 8546
```

And finally `sudo ufw enable`. Adjust the policy to your own needs if you already had ufw running.

## Connect Metamask to Sepolia

In Metamask, go to Settings, Network, Add Network and create a new custom network. 

Call it `Sepolia Test Network`, set the New RPC URL to your Sepolia Geth as `http://IP-OF-GETH-BOX:8545`, Chain ID `11155111`, currency symbol `ETH` and leave Explorer blank. Until there is a Sepolia Explorer, that is.

If this fails, you might not have added the `--http.corsdomain=*` to `geth.yml`.

## Windows mining

> Your AntiVirus and browser will likely quarantine these as cryptominers, because they are. You'll need to allow-list them.

Grab [ethminer](https://github.com/ethereum-mining/ethminer/releases). It's open source, but doesn't work with newer-generation cards.

Extract that somewhere, get into a PowerShell or create a bat file, then start it like so: For NVidia, `.\ethminer.exe -U -P getwork://IP-OF-GETH-BOX:8545` or for AMD, `.\ethminer.exe -G -P getwork://IP-OF-GETH-BOX:8545`.

You can get help with `.\ethminer.exe --help`.

An alternative is [t-rex](https://github.com/trexminer/T-Rex/releases), which is closed-source. It takes a 1% dev cut, and supports newer-generation cards. Download it, then run it in PowerShell or via bat as `.\t-rex.exe -a ethash -o http://IP-OF-GETH-BOX:8545`.

You can get help with `.\t-rex.exe --help`.

In my testing both ethminer and t-rex get almost the same hashrate from GTX 1080 and GTX 1070, with T-Rex ahead by maybe 5%. T-Rex does have a nice UI.

## Done

In your Geth logs, you should see `🔨 mined potential block` and `🔗 block reached canonical chain` as well as the occasional
`⑂ block became an uncle`, and in ethminer you should see `Job:` and `**Accepted` messages.

Enjoy the feeling of early Ethereum mining. Difficulty on Sepolia is low, you should see rewards come in steadily.
