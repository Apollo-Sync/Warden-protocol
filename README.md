# Warden-protocol

Update system

    sudo apt update
    sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y


    
Install Go

    rm -rf $HOME/go
    sudo rm -rf /usr/local/go
    cd $HOME
    curl https://dl.google.com/go/go1.20.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
    cat <<'EOF' >>$HOME/.profile
    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export GO111MODULE=on
    export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
    EOF
    source $HOME/.profile
    go version



Installation & Configuration

a. Build the wardend binary and initalize the chain home folder: replace <custom_moniker> by your monkier

    git clone --depth 1 --branch v0.3.0 https://github.com/warden-protocol/wardenprotocol/
    
    cd wardenprotocol
    
    make build-wardend

    cd build
    
    sudo cp wardend /usr/local/bin/
    
    wardend version
    
    wardend init <custom_moniker>
    
b. Prepare the genesis file:

    cd $HOME/.warden/config
    rm genesis.json
    wget https://raw.githubusercontent.com/warden-protocol/networks/main/testnets/buenavista/genesis.json

c. And set some mandatory configuration options:

    # Set minimum gas price & peers
    sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.0025uward"/' app.toml
    sed -i 's/persistent_peers = ""/persistent_peers = "ddb4d92ab6eba8363bab2f3a0d7fa7a970ae437f@sentry-1.buenavista.wardenprotocol.org:26656,c717995fd56dcf0056ed835e489788af4ffd8fe8@sentry-2.buenavista.wardenprotocol.org:26656,e1c61de5d437f35a715ac94b88ec62c482edc166@sentry-3.buenavista.wardenprotocol.org:26656"/' config.toml


d. Add Peer:

    SEEDS="8288657cb2ba075f600911685670517d18f54f3b@warden-testnet-seed.itrocket.net:18656"
    PEERS="b14f35c07c1b2e58c4a1c1727c89a5933739eeea@warden-testnet-peer.itrocket.net:18656,61446070887838944c455cb713a7770b41f35ac5@37.60.249.101:26656,0be8cf6de2a01a6dc7adb29a801722fe4d061455@65.109.115.100:27060,8288657cb2ba075f600911685670517d18f54f3b@65.108.231.124:18656,dc0122e37c203dec43306430a1f1879650653479@37.27.97.16:26656,6fb5cf2179ca9dd98ababd1c8d29878b2021c5c3@146.19.24.175:26856"
    sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.warden/config/config.toml
    

e. Star Warden:

    wardend start

f. Check sync: false

    wardend status 2>&1 | jq .sync_info

# Create a validator
If you want to create a validator in the testnet, follow the instructions in the Creating a validator section.

Create Wallet

    wardend keys add wallet

Faucet token: replace < your-address> by your warden address

    curl -XPOST -d '{"address": "<your-address>"}' https://faucet.buenavista.wardenprotocol.org

Check Banlance

    wardend q bank balances $(wardend keys show wallet -a)

# Create a new validator
Once the node is synced and you have the required WARD, you can become a validator.

To create a validator and initialize it with a self-delegation, you need to create a validator.json file and submit a create-validator transaction.

Obtain your validator public key by running the following command:

    wardend comet show-validator

The output will be similar to this (with a different key):

    {"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="}

Create validator.json file

    nano validator.json

The validator.json file has the following format: Change your personal information accordingly

    {    
    "pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="},
    "amount": "1000000uward",
    "moniker": "your-node-moniker",
    "identity": "Dzinlab validator",
    "website": "optional website for your validator",
    "security": "optional security contact for your validator",
    "details": "optional details for your validator",
    "commission-rate": "0.1",
    "commission-max-rate": "0.2",
    "commission-max-change-rate": "0.01",
    "min-self-delegation": "1"
    }

Finally, we're ready to submit the transaction to create the validator:

    wardend tx staking create-validator $HOME/validator.json \
    --from=wallet \
    --chain-id=buenavista-1 \
    --fees=500uward

Explorer

    https://explorer.nodesync.top/Warden-Testnet/staking


Delegate Token to your own validator

        wardend tx staking delegate $(wardend keys show wallet --bech val -a)  1000000uward \
        --from=wallet \
        --chain-id=buenavista-1 \
        --fees=500uward

Withdraw rewards and commission from your validator

        wardend tx distribution withdraw-rewards $(wardend keys show wallet --bech val -a) \
        --from wallet \
        --commission \
        --chain-id=buenavista-1 \
        --fees=500uward

Unjail validator

        wardend tx slashing unjail \
        --from wallet \
        --commission \
        --chain-id=buenavista-1 \
        --fees=500uward

Services Management

        # Reload Service
        sudo systemctl daemon-reload
        # Enable Service
        sudo systemctl enable wardend
        # Disable Service
        sudo systemctl disable wardend
        # Start Service
        sudo systemctl start wardend
        # Stop Service
        sudo systemctl stop wardend
        # Restart Service
        sudo systemctl restart wardend
        # Check Service Status
        sudo systemctl status wardend
        # Check Service Logs
        sudo journalctl -u wardend -f --no-hostname -o cat

 Backup Validator

         cat $HOME/.warden/config/priv_validator_key.json

Remove node

        sudo systemctl stop wardend && sudo systemctl disable wardend && \
        sudo rm /etc/systemd/system/wardend.service && sudo systemctl daemon-reload && \
        rm -rf $HOME/.warden && \
        rm -rf $HOME/wardenprotocol && \
        rm -rf $HOME/warden_auto && \
        rm -f $(which wardend) && \
        rm -rf $HOME/warden
  # DONE 
    

        
