# Pi-Relay

## Updating your topology files

### Create topologyUpdater.sh

```bash
nano $HOME/.local/bin
```

{% hint style="warning" %}
Change CNODE\_PORT to match the port your relay is running on.
{% endhint %}

```bash
#!/bin/bash
# shellcheck disable=SC2086,SC2034
 
USERNAME=$(whoami)
CNODE_PORT=3003 # must match your relay node port as set in the startup command
CNODE_HOSTNAME="CHANGE ME"  # optional. must resolve to the IP you are requesting from
CNODE_BIN="/home/ada/.local/bin"
CNODE_HOME=$NODE_HOME
CNODE_LOG_DIR="\${CNODE_HOME}/logs"
GENESIS_JSON="\${CNODE_HOME}/${NODE_CONFIG}-shelley-genesis.json"
NETWORKID=\$(jq -r .networkId \$GENESIS_JSON)
CNODE_VALENCY=1   # optional for multi-IP hostnames
NWMAGIC=\$(jq -r .networkMagic < \$GENESIS_JSON)
[[ "\${NETWORKID}" = "Mainnet" ]] && HASH_IDENTIFIER="--mainnet" || HASH_IDENTIFIER="--testnet-magic \${NWMAGIC}"
[[ "\${NWMAGIC}" = "764824073" ]] && NETWORK_IDENTIFIER="--mainnet" || NETWORK_IDENTIFIER="--testnet-magic \${NWMAGIC}"
 
export PATH="\${CNODE_BIN}:\${PATH}"
export CARDANO_NODE_SOCKET_PATH="\${CNODE_HOME}/db/socket"
 
blockNo=\$(/home/ada/.local/bin/cardano-cli query tip \${NETWORK_IDENTIFIER} | jq -r .blockNo )
 
# Note:
# if you run your node in IPv4/IPv6 dual stack network configuration and want announced the
# IPv4 address only please add the -4 parameter to the curl command below  (curl -4 -s ...)
if [ "\${CNODE_HOSTNAME}" != "CHANGE ME" ]; then
  T_HOSTNAME="&hostname=\${CNODE_HOSTNAME}"
else
  T_HOSTNAME=''
fi

if [ ! -d \${CNODE_LOG_DIR} ]; then
  mkdir -p \${CNODE_LOG_DIR};
fi
 
curl -4 -s "https://api.clio.one/htopology/v1/?port=\${CNODE_PORT}&blockNo=\${blockNo}&valency=\${CNODE_VALENCY}&magic=\${NWMAGIC}\${T_HOSTNAME}" | tee -a \$CNODE_LOG_DIR/topologyUpdater_lastresult.json
```

Make it executable & run it once.

```bash
cd $HOME
chmod +x topologyUpdater.sh
./topologyUpdater.sh
```

If successful it will print. 

> `{ "resultcode": "201", "datetime":"2021-03-29 01:23:45", "clientIp": "1.2.3.4", "iptype": 4, "msg": "nice to meet you" }`

Schedule topology updater to run once every hour. Open cron and choose nano as your editor if it asks.

```bash
crontab -l
```

Paste the following in, save & exit.

Complete this section after **four hours** when your relay node IP is properly registered.

```bash
33 * * * * ${HOME}/.local/bin/topologyUpdater.sh
```

{% hint style="success" %}
After four hours and four updates, your node IP will be registered in the topology fetch list.
{% endhint %}

### Relay topology pull

{% hint style="danger" %}
Complete this section after **four hours** when your relay node IP is properly registered.
{% endhint %}

Create `relay-topology_pull.sh` script which fetches your relay node buddies and updates your topology file. **Update with your block producer's private ipv4 address.**

```bash
#!/bin/bash
BLOCKPRODUCING_IP=<BLOCK PRODUCERS PRIVATE IP ADDRESS>
BLOCKPRODUCING_PORT=3000
curl -s -o $NODE_FILES/${NODE_CONFIG}-topology.json "https://api.clio.one/htopology/v1/fetch/?max=15&customPeers=\${BLOCKPRODUCING_IP}:\${BLOCKPRODUCING_PORT}:1|relays-new.cardano-mainnet.iohk.io:3001:2"
```

Restart the cardano-service for changes to take effect & check that it is running.

```bash
cardano-service restart
cardano-service status
```



