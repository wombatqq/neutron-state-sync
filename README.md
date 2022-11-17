# RPC node & State sync endpoint for Neutron Quark Testnet
# RPC node

RPC endpoint with "default" pruning: http://38.242.235.176:26657/ (hosted by wombat#7690)

# State sync guide

1. Follow official documentation in order to setup a full node in Neutron Testnet: https://github.com/neutron-org/testnets/tree/main/quark.
Make sure you use the actual version of ``neutrond``.
2. Set up a service to run ``neutrond`` binary.
3. Make sure it is stopped and reset database:
```
sudo systemctl stop neutrond.service
neutrond tendermint unsafe-reset-all --home $HOME/.neutrond
```
4. Use variables to connect to RPC node and set the number and the hash of the block for state sync:
```
RPC_PEER="8bf5dd948e3d4a482ddc80d7746a56b3706bcd28@38.242.235.176:26656"
SNAP_RPC="http://38.242.235.176:26657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
TRUST_HEIGHT=$((LATEST_HEIGHT - 1000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$TRUST_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$TRUST_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.neutrond/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$RPC_PEER\"/" $HOME/.neutrond/config/config.toml
```
5. Run the service and wait for a few minutes until snaphost is fetched and applied:
```
sudo systemctl restart neutrond.service && sudo journalctl -u neutrond.service -f -o cat
```
