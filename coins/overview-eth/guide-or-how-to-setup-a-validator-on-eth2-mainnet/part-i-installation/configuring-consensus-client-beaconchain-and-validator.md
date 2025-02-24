# Configuring consensus client (beacon chain and validator)

Your choice of [Lighthouse](https://github.com/sigp/lighthouse), [Nimbus](https://github.com/status-im/nimbus-eth2), [Teku](https://consensys.net/knowledge-base/ethereum-2/teku/), [Prysm](https://github.com/prysmaticlabs/prysm) or [Lodestar](https://lodestar.chainsafe.io).

{% tabs %}
{% tab title="Lighthouse" %}
{% hint style="info" %}
[Lighthouse](https://github.com/sigp/lighthouse) is an Eth client with a heavy focus on speed and security. The team behind it, [Sigma Prime](https://sigmaprime.io), is an information security and software engineering firm who have funded Lighthouse along with the Ethereum Foundation, Consensys, and private individuals. Lighthouse is built in Rust and offered under an Apache 2.0 License.
{% endhint %}



:gear: **4.1. Install rust dependency**

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Enter '1' to proceed with the default install.

Update your environment variables.

```bash
echo export PATH="$HOME/.cargo/bin:$PATH" >> ~/.bashrc
source ~/.bashrc
```

Install rust dependencies.

```
sudo apt-get update
sudo apt install -y git gcc g++ make cmake pkg-config libssl-dev libclang-dev clang
```



:bulb: **4.2. Build Lighthouse from source**

```bash
mkdir ~/git
cd ~/git
git clone https://github.com/sigp/lighthouse.git
cd lighthouse
git fetch --all && git checkout stable && git pull
make
```

{% hint style="info" %}
In case of compilation errors, run the following sequence.

```
rustup update
cargo clean
make
```
{% endhint %}

{% hint style="info" %}
This build process may take a few minutes.
{% endhint %}

Verify lighthouse was installed properly by checking the version number.

```
lighthouse --version
```



:tophat: **4.3. Import validator key**

{% hint style="info" %}
When you import your keys into Lighthouse, your validator signing key(s) are stored in the `$HOME/.lighthouse/mainnet/validators` folder.
{% endhint %}

Run the following command to import your validator keys from the eth2deposit-cli tool directory.

Enter your **keystore password** to import accounts.

```bash
lighthouse account validator import --network mainnet --directory=$HOME/eth2deposit-cli/validator_keys
```

Verify the accounts were imported successfully.

```bash
lighthouse account_manager validator list --network mainnet
```



{% hint style="danger" %}
**WARNING**: DO NOT USE THE ORIGINAL KEYSTORES TO VALIDATE WITH ANOTHER CLIENT, OR YOU WILL GET SLASHED.
{% endhint %}



:fire: **4.4. Configure port forwarding and/or firewall**

Specific to your networking setup or cloud provider settings, [ensure your validator's firewall ports are open and reachable.](../../guide-or-security-best-practices-for-a-eth2-validator-beaconchain-node.md#configure-your-firewall)

* **Lighthouse consensus client** requires port 9000 for tcp and udp
* **Execution client** requires port 30303 for tcp and udp

{% hint style="info" %}
:sparkles: **Port Forwarding Tip:** You'll need to forward and open ports to your validator. Verify it's working with [https://www.yougetsignal.com/tools/open-ports/](https://www.yougetsignal.com/tools/open-ports/) or [https://canyouseeme.org/](https://canyouseeme.org) .
{% endhint %}



:chains: **4.5. Start the beacon chain**

:cake: **Benefits of using systemd for your beacon chain**

1. Auto-start your beacon chain when the computer reboots due to maintenance, power outage, etc.
2. Automatically restart crashed beacon chain processes.
3. Maximize your beacon chain up-time and performance.

:tools: **Setup Instructions for Systemd**

Run the following to create a **unit file** to define your`beacon-chain.service` configuration. Simply copy and paste.

```bash
cat > $HOME/beacon-chain.service << EOF 
# The eth beacon chain service (part of systemd)
# file: /etc/systemd/system/beacon-chain.service 

[Unit]
Description     = eth beacon chain service
Wants           = network-online.target
After           = network-online.target 

[Service]
User            = $USER
ExecStart       = $(which lighthouse) bn --staking --validator-monitor-auto --metrics --network mainnet
Restart         = on-failure

[Install]
WantedBy    = multi-user.target
EOF
```



{% hint style="info" %}
:fire: **Lighthouse Pro Tip**: On the **ExecStart** line, adding the `--eth1-endpoints` flag allows for redundant execution clients. Separate with comma. Make sure the endpoint does not end with a trailing slash or`/` Remove it.

```bash
# Example:
--eth1-endpoints http://localhost:8545,https://nodes.mewapi.io/rpc/eth,https://mainnet.eth.cloud.ava.do,https://mainnet.infura.io/v3/xxx
```

:money\_with\_wings: Find free ethereum fallback nodes at [https://ethereumnodes.com/](https://ethereumnodes.com)
{% endhint %}



Move the unit file to `/etc/systemd/system`

```bash
sudo mv $HOME/beacon-chain.service /etc/systemd/system/beacon-chain.service
```

Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/beacon-chain.service
```

Run the following to enable auto-start at boot time and then start your beacon node service.

```
sudo systemctl daemon-reload
sudo systemctl enable beacon-chain
sudo systemctl start beacon-chain
```



{% hint style="info" %}
**Troubleshooting common issues**:

_The beacon chain couldn't connect to the :8545 service?_

* In the beacon chain unit file under \[Service], add, "`ExecStartPre = /bin/sleep 30`" so that it waits 30 seconds for execution client to startup before connecting.

_CRIT Invalid eth1 chain id. Please switch to correct chain id._

* Allow your execution client to fully sync to mainnet.
{% endhint %}



{% hint style="success" %}
Nice work. Your beacon chain is now managed by the reliability and robustness of systemd. Below are some commands for using systemd.
{% endhint %}



:tools: **Some helpful systemd commands**

***

**🗄 Viewing and filtering logs**

```bash
#view and follow the log
journalctl --unit=beacon-chain -f
```

```bash
#view log since yesterday
journalctl --unit=beacon-chain --since=yesterday
```

```bash
#view log since today
journalctl --unit=beacon-chain --since=today
```

```bash
#view log between a date
journalctl --unit=beacon-chain --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```

:mag\_right: **View the status of the beacon chain**

```
sudo systemctl status beacon-chain
```

:arrows\_counterclockwise: **Restarting the beacon chain**

```
sudo systemctl reload-or-restart beacon-chain
```

:octagonal\_sign: **Stopping the beacon chain**

```
sudo systemctl stop beacon-chain
```



:dna: **4.6. Start the validator**

:rocket: **Setup Graffiti**

Setup your `graffiti`, a custom message included in blocks your validator successfully proposes. Add optional graffiti between the single quotes.

```bash
MY_GRAFFITI=''
# Examples
# MY_GRAFFITI='poapAAAAACGatUA1bLuDnL4FMD13BfoD'
# MY_GRAFFITI='eth rulez!'
```

:cake: **Benefits of using systemd for your validator**

1. Auto-start your validator when the computer reboots due to maintenance, power outage, etc.
2. Automatically restart crashed validator processes.
3. Maximize your validator up-time and performance.

:tools: **Setup Instructions for Systemd**

Run the following to create a **unit file** to define your`validator.service` configuration. Simply copy and paste.

```bash
cat > $HOME/validator.service << EOF 
# The eth validator service (part of systemd)
# file: /etc/systemd/system/validator.service 

[Unit]
Description     = eth validator service
Wants           = network-online.target beacon-chain.service
After           = network-online.target 

[Service]
User            = $USER
ExecStart       = $(which lighthouse) vc --network mainnet --graffiti "${MY_GRAFFITI}" --metrics --enable-doppelganger-protection 
Restart         = on-failure

[Install]
WantedBy    = multi-user.target
EOF
```

Move the unit file to `/etc/systemd/system`

```bash
sudo mv $HOME/validator.service /etc/systemd/system/validator.service
```

Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/validator.service
```

Run the following to enable auto-start at boot time and then start your validator.

```
sudo systemctl daemon-reload
sudo systemctl enable validator
sudo systemctl start validator
```

{% hint style="success" %}
Nice work. Your validator is now managed by the reliability and robustness of systemd. Below are some commands for using systemd.
{% endhint %}



**🛠 Some helpful systemd commands**

**🗄 Viewing and filtering logs**

```bash
#view and follow the log
journalctl --unit=validator -f
```

```bash
#view log since yesterday
journalctl --unit=validator --since=yesterday
```

```bash
#view log since today
journalctl --unit=validator --since=today
```

```bash
#view log between a date
journalctl --unit=validator --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```

**🔎 View the status of the validator**

```
sudo systemctl status validator
```

**🔄 Restarting the validator**

```
sudo systemctl reload-or-restart validator
```

**🛑 Stopping the validator**

```
sudo systemctl stop validator
```
{% endtab %}

{% tab title="Nimbus" %}
{% hint style="info" %}
[Nimbus](https://our.status.im/tag/nimbus/) is a research project and a client implementation for Ethereum 2.0 designed to perform well on embedded systems and personal mobile devices, including older smartphones with resource-restricted hardware. The Nimbus team are from [Status](https://status.im/about/) the company best known for [their messaging app/wallet/Web3 browser](https://status.im) by the same name. Nimbus (Apache 2) is written in Nim, a language with Python-like syntax that compiles to C.
{% endhint %}



{% hint style="info" %}
:bulb: **Noteworthy**: binaries for all the usual platforms as well as dockers for x86 and arm can be found below:

[https://github.com/status-im/nimbus-eth2/releases/](https://github.com/status-im/nimbus-eth2/releases/)\
[https://hub.docker.com/r/statusim/nimbus-eth2](https://hub.docker.com/r/statusim/nimbus-eth2)
{% endhint %}



:gear: **4.1. Build Nimbus from source**

Install dependencies.

```
sudo apt-get update
sudo apt-get install curl build-essential git -y
```

Install and build Nimbus.

```bash
mkdir ~/git 
cd ~/git
git clone https://github.com/status-im/nimbus-eth2
cd nimbus-eth2
make nimbus_beacon_node
```

{% hint style="info" %}
The build process may take a few minutes.
{% endhint %}

Verify Nimbus was installed properly by displaying the help.

```bash
cd $HOME/git/nimbus-eth2/build
./nimbus_beacon_node --help
```

Copy the binary file to `/usr/bin`

```bash
sudo cp $HOME/git/nimbus-eth2/build/nimbus_beacon_node /usr/bin
```



:tophat: **4.2. Import validator key**

Create a directory structure to store nimbus data.

```bash
sudo mkdir -p /var/lib/nimbus
```

Take ownership of this directory and set the correct permission level.

```bash
sudo chown $(whoami):$(whoami) /var/lib/nimbus
sudo chmod 700 /var/lib/nimbus
```

The following command will import your validator keys.

Enter your **keystore password** to import accounts.

```bash
cd $HOME/git/nimbus-eth2
build/nimbus_beacon_node deposits import --data-dir=/var/lib/nimbus $HOME/eth2deposit-cli/validator_keys
```

Now you can verify the accounts were imported successfully by doing a directory listing.

```bash
ll /var/lib/nimbus/validators
```

You should see a folder named for each of your validator's pubkey.

{% hint style="info" %}
When you import your keys into Nimbus, your validator signing key(s) are stored in the `/var/lib/nimbus` folder, under `secrets` and `validators.`

The `secrets` folder contains the common secret that gives you access to all your validator keys.

The `validators` folder contains your signing keystore(s) (encrypted keys). Keystores are used by validators as a method for exchanging keys.

For more on keys and keystores, see [here](https://blog.ethereum.org/2020/05/21/keys/).
{% endhint %}



{% hint style="danger" %}
**WARNING**: DO NOT USE THE ORIGINAL KEYSTORES TO VALIDATE WITH ANOTHER CLIENT, OR YOU WILL GET SLASHED.
{% endhint %}



:fire: **4.3. Configure port forwarding and/or firewall**

Specific to your networking setup or cloud provider settings, [ensure your validator's firewall ports are open and reachable.](../../guide-or-security-best-practices-for-a-eth2-validator-beaconchain-node.md#configure-your-firewall)

* **Nimbus consensus client** will use port 9000 for tcp and udp
* **Execution client** requires port 30303 for tcp and udp

{% hint style="info" %}
:sparkles: **Port Forwarding Tip:** You'll need to forward and open ports to your validator. Verify it's working with [https://www.yougetsignal.com/tools/open-ports/](https://www.yougetsignal.com/tools/open-ports/) or [https://canyouseeme.org/](https://canyouseeme.org) .
{% endhint %}



:snowboarder: **4.4. Start the beacon chain and validator**

{% hint style="info" %}
Nimbus combines both the beacon chain and validator into one process.
{% endhint %}

:rocket: **Setup Graffiti**

***

Setup your `graffiti`, a custom message included in blocks your validator successfully proposes. Add optional graffiti between the single quotes.

```bash
MY_GRAFFITI=''
# Examples
# MY_GRAFFITI='poapAAAAACGatUA1bLuDnL4FMD13BfoD'
# MY_GRAFFITI='eth rulez!'
```

**🍰 Benefits of using systemd for your beacon chain and validator**

1. Auto-start your beacon chain when the computer reboots due to maintenance, power outage, etc.
2. Automatically restart crashed beacon chain processes.
3. Maximize your beacon chain up-time and performance.

**🛠 Setup Instructions**

Run the following to create a **unit file** to define your`beacon-chain.service` configuration. Simply copy and paste.

```bash
cat > $HOME/beacon-chain.service << EOF 
# The eth2 beacon chain service (part of systemd)
# file: /etc/systemd/system/beacon-chain.service 

[Unit]
Description     = eth2 beacon chain service
Wants           = network-online.target
After           = network-online.target 

[Service]
Type            = simple
User            = $(whoami)
WorkingDirectory= /var/lib/nimbus
ExecStart       = /bin/bash -c '/usr/bin/nimbus_beacon_node --network=mainnet --graffiti="${MY_GRAFFITI}" --data-dir=/var/lib/nimbus --web3-url=ws://127.0.0.1:8546 --metrics --metrics-port=8008 --rpc --rpc-port=9091 --validators-dir=/var/lib/nimbus/validators --secrets-dir=/var/lib/nimbus/secrets --log-file=/var/lib/nimbus/beacon.log'
Restart         = on-failure

[Install]
WantedBy    = multi-user.target
EOF
```

{% hint style="warning" %}
Nimbus only supports websocket connections ("ws://" and "wss://") for the ETH1 node. Geth, OpenEthereum and Infura ETH1 nodes are verified compatible.
{% endhint %}

Move the unit file to `/etc/systemd/system`

```bash
sudo mv $HOME/beacon-chain.service /etc/systemd/system/beacon-chain.service
```

Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/beacon-chain.service
```

Run the following to enable auto-start at boot time and then start your beacon node service.

```
sudo systemctl daemon-reload
sudo systemctl enable beacon-chain
sudo systemctl start beacon-chain
```

{% hint style="success" %}
Nice work. Your beacon chain is now managed by the reliability and robustness of systemd. Below are some commands for using systemd.
{% endhint %}



:tools: **Some helpful systemd commands**

**🗄 Viewing and filtering logs**

```bash
#view and follow the log
journalctl --unit=beacon-chain -f
```

```bash
#view log since yesterday
journalctl --unit=beacon-chain --since=yesterday
```

```bash
#view log since today
journalctl --unit=beacon-chain --since=today
```

```bash
#view log between a date
journalctl --unit=beacon-chain --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```

:mag\_right: **View the status of the beacon chain**

```
sudo systemctl status beacon-chain
```

:arrows\_counterclockwise: **Restarting the beacon chain**

```
sudo systemctl reload-or-restart beacon-chain
```

:octagonal\_sign: **Stopping the beacon chain**

```
sudo systemctl stop beacon-chain
```
{% endtab %}

{% tab title="Teku" %}
{% hint style="info" %}
[PegaSys Teku](https://consensys.net/knowledge-base/ethereum-2/teku/) (formerly known as Artemis) is a Java-based Ethereum 2.0 client designed & built to meet institutional needs and security requirements. PegaSys is an arm of [ConsenSys](https://consensys.net) dedicated to building enterprise-ready clients and tools for interacting with the core Ethereum platform. Teku is Apache 2 licensed and written in Java, a language notable for its maturity & ubiquity.
{% endhint %}



:gear: **4.1 Build Teku from source**

Install git.

```
sudo apt-get install git -y
```

Install Java 11.

For **Ubuntu 20.x**, use the following

```
sudo apt update
sudo apt install openjdk-11-jdk -y
```

Verify Java 11+ is installed.

```bash
java --version
```

Install and build Teku.

```bash
mkdir ~/git
cd ~/git
git clone https://github.com/ConsenSys/teku.git
cd teku
./gradlew distTar installDist
```



{% hint style="info" %}
This build process may take a few minutes.
{% endhint %}



Verify Teku was installed properly by displaying the version.

```bash
cd $HOME/git/teku/build/install/teku/bin
./teku --version
```

Copy the teku binary file to `/usr/bin/teku`

```bash
sudo cp -r $HOME/git/teku/build/install/teku /usr/bin/teku
```



:fire: **4.2. Configure port forwarding and/or firewall**

Specific to your networking setup or cloud provider settings, [ensure your validator's firewall ports are open and reachable.](../../guide-or-security-best-practices-for-a-eth2-validator-beaconchain-node.md#configure-your-firewall)

* **Teku consensus client** will use port 9000 for tcp and udp
* **Execution client** requires port 30303 for tcp and udp

{% hint style="info" %}
:sparkles: **Port Forwarding Tip**: You'll need to forward and open ports to your validator. Verify it's working with [https://www.yougetsignal.com/tools/open-ports/](https://www.yougetsignal.com/tools/open-ports/) or [https://canyouseeme.org/](https://canyouseeme.org) .
{% endhint %}



:snowboarder: **4.3. Configure the beacon chain and validator**

{% hint style="info" %}
Teku combines both the beacon chain and validator into one process.
{% endhint %}

Setup a directory structure for Teku.

```bash
sudo mkdir -p /var/lib/teku
sudo mkdir -p /etc/teku
sudo chown $(whoami):$(whoami) /var/lib/teku
```

Copy your `validator_files` directory to the data directory we created above and remove the extra deposit\_data file.

```bash
cp -r $HOME/eth2deposit-cli/validator_keys /var/lib/teku
rm /var/lib/teku/validator_keys/deposit_data*
```

{% hint style="danger" %}
**WARNING**: DO NOT USE THE ORIGINAL KEYSTORES TO VALIDATE WITH ANOTHER CLIENT, OR YOU WILL GET SLASHED.
{% endhint %}

Storing your **keystore password** in a text file is required so that Teku can decrypt and load your validators automatically.

Update `my_keystore_password_goes_here` with your **keystore password** between the single quotation marks and then run the command to save it to validators-password.txt

```bash
echo 'my_keystore_password_goes_here' > $HOME/validators-password.txt
```

Confirm that your **keystore password** is correct.

```bash
cat $HOME/validators-password.txt
```

Move the password file and make it read-only.

```bash
sudo mv $HOME/validators-password.txt /etc/teku/validators-password.txt
sudo chmod 600 /etc/teku/validators-password.txt
```

Clear the bash history in order to remove traces of keystore password.

```bash
shred -u ~/.bash_history && touch ~/.bash_history
```

:rocket: **Setup Graffiti**

Setup your `graffiti`, a custom message included in blocks your validator successfully proposes. Add optional graffiti between the single quotes.

```bash
MY_GRAFFITI=''
# Examples
# MY_GRAFFITI='poapAAAAACGatUA1bLuDnL4FMD13BfoD'
# MY_GRAFFITI='eth rulez!'
```

:fast\_forward: **Setup Teku Checkpoint Sync**

{% hint style="info" %}
Teku's Checkpoint Sync utilizes Infura to create the fastest syncing Ethereum beacon chain.
{% endhint %}

1\. Sign up for [a free infura account](https://infura.io/register).

2\. Create a project.

![](../../../../.gitbook/assets/inf1.png)

3\. Add a project name and save changes.

4\. Copy your Project's ENDPOINT. Ensure the correct Network is selected with the dropdown box.

![](../../../../.gitbook/assets/inf2.png)

Replace `<my infura Project's ENDPOINT>` with your Infura endpoint and then run the following command to set the `INFURA_PROJECT_ENDPOINT` variable.

```bash
INFURA_PROJECT_ENDPOINT=<my Infura Project's ENDPOINT>
```

```bash
# Example
# INFURA_PROJECT_ENDPOINT=https://1Rjimg6q8hxGaRfxmEf9vxyBEk5n:c42acfe90bcae227f9ec19b22e733550@eth2-beacon-mainnet.infura.io
```

Confirm that your Infura Project Endpoint looks correct.

```bash
echo $INFURA_PROJECT_ENDPOINT
```

Generate your Teku Config file. Simply copy and paste.

```bash
cat > $HOME/teku.yaml << EOF
# network
network: "mainnet"
initial-state: "${INFURA_PROJECT_ENDPOINT}/eth/v2/debug/beacon/states/finalized" 

# p2p
p2p-enabled: true
p2p-port: 9000

# validators
validator-keys: "/var/lib/teku/validator_keys:/var/lib/teku/validator_keys"
validators-graffiti: "${MY_GRAFFITI}"

# Eth 1
eth1-endpoint: "http://localhost:8545"

# metrics
metrics-enabled: true
metrics-port: 8008

# database
data-path: "/var/lib/teku"
data-storage-mode: "prune"

# rest api
rest-api-port: 5051
rest-api-docs-enabled: true
rest-api-enabled: true

# logging
log-include-validator-duties-enabled: true
log-destination: CONSOLE
EOF
```

Move the config file to `/etc/teku`

```bash
sudo mv $HOME/teku.yaml /etc/teku/teku.yaml
```



:tophat: **4.4 Import validator key**

{% hint style="info" %}
When specifying directories for your validator-keys, Teku expects to find identically named keystore and password files.

For example `keystore-m_12221_3600_1_0_0-11222333.json` and `keystore-m_12221_3600_1_0_0-11222333.txt`
{% endhint %}

Create a corresponding password file for every one of your validators.

```bash
for f in /var/lib/teku/validator_keys/keystore*.json; do cp /etc/teku/validators-password.txt /var/lib/teku/validator_keys/$(basename $f .json).txt; done
```

Verify that your validator's keystore and validator's passwords are present by checking the following directory.

```bash
ll /var/lib/teku/validator_keys
```



:checkered\_flag: **4.5. Start the beacon chain and validator**

Use **systemd** to manage starting and stopping teku.

**🍰 Benefits of using systemd for your beacon chain and validator**

1. Auto-start your beacon chain when the computer reboots due to maintenance, power outage, etc.
2. Automatically restart crashed beacon chain processes.
3. Maximize your beacon chain up-time and performance.

:tools: **Setup Instructions**

***

Run the following to create a **unit file** to define your`beacon-chain.service` configuration. Simply copy and paste.

```bash
cat > $HOME/beacon-chain.service << EOF
# The eth2 beacon chain service (part of systemd)
# file: /etc/systemd/system/beacon-chain.service 

[Unit]
Description     = eth2 beacon chain service
Wants           = network-online.target
After           = network-online.target 

[Service]
User            = $(whoami)
ExecStart       = /usr/bin/teku/bin/teku -c /etc/teku/teku.yaml
Restart         = on-failure
Environment     = JAVA_OPTS=-Xmx5g

[Install]
WantedBy	= multi-user.target
EOF
```

Move the unit file to `/etc/systemd/system`

```bash
sudo mv $HOME/beacon-chain.service /etc/systemd/system/beacon-chain.service
```

Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/beacon-chain.service
```

Run the following to enable auto-start at boot time and then start your beacon node service.

```
sudo systemctl daemon-reload
sudo systemctl enable beacon-chain
sudo systemctl start beacon-chain
```

{% hint style="success" %}
Nice work. Your beacon chain is now managed by the reliability and robustness of systemd. Below are some commands for using systemd.
{% endhint %}

:tools: **Some helpful systemd commands**

**🗄 Viewing and filtering logs**

```bash
#view and follow the log
journalctl --unit=beacon-chain -f
```

```bash
#view log since yesterday
journalctl --unit=beacon-chain --since=yesterday
```

```bash
#view log since today
journalctl --unit=beacon-chain --since=today
```

```bash
#view log between a date
journalctl --unit=beacon-chain --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```

:mag\_right: **View the status of the beacon chain**

```
sudo systemctl status beacon-chain
```

:arrows\_counterclockwise: **Restarting the beacon chain**

```
sudo systemctl reload-or-restart beacon-chain
```

:octagonal\_sign: **Stopping the beacon chain**

```
sudo systemctl stop beacon-chain
```
{% endtab %}

{% tab title="Prysm" %}
{% hint style="info" %}
[Prysm](https://github.com/prysmaticlabs/prysm) is a Go implementation of Ethereum 2.0 protocol with a focus on usability, security, and reliability. Prysm is developed by [Prysmatic Labs](https://prysmaticlabs.com), a company with the sole focus on the development of their client. Prysm is written in Go and released under a GPL-3.0 license.
{% endhint %}



:gear: **4.1. Install Prysm**

```bash
mkdir ~/prysm && cd ~/prysm 
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh 
```



:fire: **4.2. Configure port forwarding and/or firewall**

Specific to your networking setup or cloud provider settings, [ensure your validator's firewall ports are open and reachable.](../../guide-or-security-best-practices-for-a-eth2-validator-beaconchain-node.md#configure-your-firewall)

* **Prysm consensus client** will use port 12000 for udp and port 13000 for tcp
* **Execution client** requires port 30303 for tcp and udp

{% hint style="info" %}
:sparkles: **Port Forwarding Tip:** You'll need to forward and open ports to your validator. Verify it's working with [https://www.yougetsignal.com/tools/open-ports/](https://www.yougetsignal.com/tools/open-ports/) or [https://canyouseeme.org/](https://canyouseeme.org) .
{% endhint %}



:tophat: **4.3. Import validator key**

Accept terms of use, accept default wallet location, enter a new **prysm-only password** to encrypt your local prysm wallet files and enter the **keystore password** for your imported accounts.

{% hint style="info" %}
If you wish, you can use the same password for the **keystore** and **prysm-only**.
{% endhint %}

```bash
$HOME/prysm/prysm.sh validator accounts import --mainnet --keys-dir=$HOME/eth2deposit-cli/validator_keys
```

Verify your validators imported successfully.

```bash
$HOME/prysm/prysm.sh validator accounts list --mainnet
```

Confirm your validator's pubkeys are listed.

> \#Example output:
>
> Showing 1 validator account View the eth1 deposit transaction data for your accounts by running \`validator accounts list --show-deposit-data
>
> Account 0 | pens-brother-heat\
> \[validating public key] 0x2374.....7121

{% hint style="danger" %}
**WARNING**: DO NOT USE THE ORIGINAL KEYSTORES TO VALIDATE WITH ANOTHER CLIENT, OR YOU WILL GET SLASHED.
{% endhint %}



:snowboarder: **4.4. Start the beacon chain**

:cake: **Benefits of using systemd for your beacon chain and validator**

1. Auto-start your beacon chain when the computer reboots due to maintenance, power outage, etc.
2. Automatically restart crashed beacon chain processes.
3. Maximize your beacon chain up-time and performance.

:tools: **Setup Instructions**

Run the following to create a **unit file** to define your`beacon-chain.service` configuration. Simply copy and paste.

```bash
cat > $HOME/beacon-chain.service << EOF 
# The eth2 beacon chain service (part of systemd)
# file: /etc/systemd/system/beacon-chain.service 

[Unit]
Description     = eth2 beacon chain service
Wants           = network-online.target
After           = network-online.target 

[Service]
Type            = simple
User            = $(whoami)
ExecStart       = $(echo $HOME)/prysm/prysm.sh beacon-chain --mainnet --p2p-max-peers=45 --http-web3provider=http://127.0.0.1:8545 --accept-terms-of-use 
Restart         = on-failure

[Install]
WantedBy    = multi-user.target
EOF
```

{% hint style="info" %}
:fire: **Prysm Pro Tip**: On the ExecStart line, adding the `--fallback-web3provider` flag allows for a backup execution client. May use flag multiple times. Make sure the endpoint does not end with a trailing slash or`/` Remove it.

```bash
--fallback-web3provider=<http://<alternate eth1 provider one> --fallback-web3provider=<http://<alternate eth1 provider two>
# Example
# --fallback-web3provider=https://nodes.mewapi.io/rpc/eth --fallback-web3provider=https://mainnet.infura.io/v3/YOUR-PROJECT-ID
```

:money\_with\_wings: Find free ethereum fallback nodes at [https://ethereumnodes.com/](https://ethereumnodes.com)
{% endhint %}

Move the unit file to `/etc/systemd/system`

```bash
sudo mv $HOME/beacon-chain.service /etc/systemd/system/beacon-chain.service
```

Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/beacon-chain.service
```

Run the following to enable auto-start at boot time and then start your beacon node service.

```
sudo systemctl daemon-reload
sudo systemctl enable beacon-chain
sudo systemctl start beacon-chain
```

{% hint style="success" %}
Nice work. Your beacon chain is now managed by the reliability and robustness of systemd. Below are some commands for using systemd.
{% endhint %}

:tools: **Some helpful systemd commands**

**🗄 Viewing and filtering logs**

```bash
#view and follow the log
journalctl --unit=beacon-chain -f
```

```bash
#view log since yesterday
journalctl --unit=beacon-chain --since=yesterday
```

```bash
#view log since today
journalctl --unit=beacon-chain --since=today
```

```bash
#view log between a date
journalctl --unit=beacon-chain --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```

:mag\_right: **View the status of the beacon chain**

```
sudo systemctl status beacon-chain
```

:arrows\_counterclockwise: **Restarting the beacon chain**

```
sudo systemctl reload-or-restart beacon-chain
```

:octagonal\_sign: **Stopping the beacon chain**

```
sudo systemctl stop beacon-chain
```



:dna: **4.5. Start the validator**

Store your **prysm-only password** in a file and make it read-only. This is required so that Prysm can decrypt and load your validators.

```bash
echo 'my_password_goes_here' > $HOME/.eth2validators/validators-password.txt
sudo chmod 600 $HOME/.eth2validators/validators-password.txt
```

Clear the bash history in order to remove traces of your **prysm-only password.**

```bash
shred -u ~/.bash_history && touch ~/.bash_history
```

:rocket: **Setup Graffiti**

Setup your `graffiti`, a custom message included in blocks your validator successfully proposes. Add optional graffiti between the single quotes.

```bash
MY_GRAFFITI=''
# Examples
# MY_GRAFFITI='poapAAAAACGatUA1bLuDnL4FMD13BfoD'
# MY_GRAFFITI='eth rulez!'
```

Run your validator automatically with systemd.

:cake: **Benefits of using systemd for your validator**

1. Auto-start your validator when the computer reboots due to maintenance, power outage, etc.
2. Automatically restart crashed validator processes.
3. Maximize your validator up-time and performance.

:tools: **Setup Instructions for systemd**

Run the following to create a **unit file** to define your`validator.service` configuration. Simply copy and paste.

```bash
cat > $HOME/validator.service << EOF 
# The eth2 validator service (part of systemd)
# file: /etc/systemd/system/validator.service 

[Unit]
Description     = eth2 validator service
Wants           = network-online.target beacon-chain.service
After           = network-online.target 

[Service]
User            = $(whoami)
ExecStart       = $(echo $HOME)/prysm/prysm.sh validator --mainnet --graffiti "${MY_GRAFFITI}" --accept-terms-of-use --wallet-password-file $(echo $HOME)/.eth2validators/validators-password.txt --enable-doppelganger
Restart         = on-failure

[Install]
WantedBy	= multi-user.target
EOF
```

Move the unit file to `/etc/systemd/system`

```bash
sudo mv $HOME/validator.service /etc/systemd/system/validator.service
```

Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/validator.service
```

Run the following to enable auto-start at boot time and then start your validator.

```
sudo systemctl daemon-reload
sudo systemctl enable validator
sudo systemctl start validator
```

:tools: **Some helpful systemd commands**

**🗄 Viewing and filtering logs**

```bash
#view and follow the log
journalctl --unit=validator -f
```

```bash
#view log since yesterday
journalctl --unit=validator --since=yesterday
```

```bash
#view log since today
journalctl --unit=validator --since=today
```

```bash
#view log between a date
journalctl --unit=validator --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```

:mag\_right: **View the status of the validator**

```
sudo systemctl status validator
```

:arrows\_counterclockwise: **Restarting the validator**

```
sudo systemctl reload-or-restart validator
```

:octagonal\_sign: **Stopping the validator**

```
sudo systemctl stop validator
```

Verify that your **validator public key** appears in the logs.

```bash
journalctl --unit=validator --since=today
# Example below
# INFO Enabled validator       voting_pubkey: 0x2374.....7121
```
{% endtab %}

{% tab title="Lodestar" %}
{% hint style="info" %}
[Lodestar ](https://lodestar.chainsafe.io)**is a Typescript implementation** of the official [Ethereum 2.0 specification](https://github.com/ethereum/eth2.0-specs) by the [ChainSafe.io](https://lodestar.chainsafe.io) team. In addition to the beacon chain client, the team is also working on 22 packages and libraries. A complete list can be found [here](https://hackmd.io/CcsWTnvRS\_eiLUajr3gi9g). Finally, the Lodestar team is leading the Eth2 space in light client research and development and has received funding from the EF and Moloch DAO for this purpose.
{% endhint %}



:gear: **4.1 Build Lodestar from source**

Install curl and git.

```bash
sudo apt-get install gcc g++ make git curl -y
```

Install yarn.

```bash
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install yarn -y
```

Confirm yarn is installed properly.

```bash
yarn --version
# Should output version >= 1.22.4
```

Install nodejs.

```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Confirm nodejs is installed properly.

```bash
nodejs -v
# Should output version >= v12.18.3
```

Install and build Lodestar.

```bash
mkdir ~/git
cd ~/git
git clone https://github.com/chainsafe/lodestar.git
cd lodestar
yarn install --ignore-optional
yarn run build
```

{% hint style="info" %}
This build process may take a few minutes.
{% endhint %}

Verify Lodestar was installed properly by displaying the help menu.

```
./lodestar --help
```



:fire: **4.2. Configure port forwarding and/or firewall**

Specific to your networking setup or cloud provider settings, [ensure your validator's firewall ports are open and reachable.](../../guide-or-security-best-practices-for-a-eth2-validator-beaconchain-node.md#configure-your-firewall)

* **Lodestar consensus client** will use port 30607 for tcp and port 9000 for udp peer discovery.
* **Execution client** requires port 30303 for tcp and udp

{% hint style="info" %}
:sparkles: **Port Forwarding Tip**: You'll need to forward and open ports to your validator. Verify it's working with [https://www.yougetsignal.com/tools/open-ports/](https://www.yougetsignal.com/tools/open-ports/) or [https://canyouseeme.org/](https://canyouseeme.org) .
{% endhint %}



:tophat: **4.3. Import validator key**

```bash
./lodestar account validator import \
  --network mainnet \
  --directory $HOME/eth2deposit-cli/validator_keys
```

Enter your **keystore password** to import accounts.

Confirm your keys were imported properly.

```
./lodestar account validator list --network mainnet
```

{% hint style="danger" %}
**WARNING**: DO NOT USE THE ORIGINAL KEYSTORES TO VALIDATE WITH ANOTHER CLIENT, OR YOU WILL GET SLASHED.
{% endhint %}



:snowboarder: **4.4. Start the beacon chain and validator**

Run the beacon chain automatically with systemd.

**🍰 Benefits of using systemd for your beacon chain**

1. Auto-start your beacon chain when the computer reboots due to maintenance, power outage, etc.
2. Automatically restart crashed beacon chain processes.
3. Maximize your beacon chain up-time and performance.

:tools: **Setup Instructions**

Run the following to create a **unit file** to define your`beacon-chain.service` configuration. Simply copy and paste.

```bash
cat > $HOME/beacon-chain.service << EOF 
# The eth2 beacon chain service (part of systemd)
# file: /etc/systemd/system/beacon-chain.service 

[Unit]
Description     = eth2 beacon chain service
Wants           = network-online.target
After           = network-online.target 

[Service]
User            = $(whoami)
WorkingDirectory= $(echo $HOME)/git/lodestar
ExecStart       = $(echo $HOME)/git/lodestar/lodestar beacon --network mainnet --eth1.providerUrl http://localhost:8545 --weakSubjectivitySyncLatest true --metrics.enabled true --metrics.serverPort 8008
Restart         = on-failure

[Install]
WantedBy	= multi-user.target
EOF
```

Move the unit file to `/etc/systemd/system`

```bash
sudo mv $HOME/beacon-chain.service /etc/systemd/system/beacon-chain.service
```

Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/beacon-chain.service
```

Run the following to enable auto-start at boot time and then start your beacon node service.

```
sudo systemctl daemon-reload
sudo systemctl enable beacon-chain
sudo systemctl start beacon-chain
```

{% hint style="success" %}
Nice work. Your beacon chain is now managed by the reliability and robustness of systemd. Below are some commands for using systemd.
{% endhint %}

:tools: **Some helpful systemd commands**

**🗄 Viewing and filtering logs**

```bash
#view and follow the log
journalctl --unit=beacon-chain -f
```

```bash
#view log since yesterday
journalctl --unit=beacon-chain --since=yesterday
```

```bash
#view log since today
journalctl --unit=beacon-chain --since=today
```

```bash
#view log between a date
journalctl --unit=beacon-chain --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```

:mag\_right: **View the status of the beacon chain**

```
sudo systemctl status beacon-chain
```

:arrows\_counterclockwise: **Restarting the beacon chain**

```
sudo systemctl reload-or-restart beacon-chain
```

:octagonal\_sign: **Stopping the beacon chain**

```
sudo systemctl stop beacon-chain
```



:dna: **4.5. Start the validator**

:rocket: **Setup Graffiti**

Setup your `graffiti`, a custom message included in blocks your validator successfully proposes. Add optional graffiti between the single quotes.

```bash
MY_GRAFFITI=''
# Examples
# MY_GRAFFITI='poapAAAAACGatUA1bLuDnL4FMD13BfoD'
# MY_GRAFFITI='eth rulez!'
```

Run the validator automatically with systemd.

**🍰 Benefits of using systemd for your validator**

1. Auto-start your validator when the computer reboots due to maintenance, power outage, etc.
2. Automatically restart crashed validator processes.
3. Maximize your validator up-time and performance.

:tools: **Setup Instructions**

Run the following to create a **unit file** to define your`validator.service` configuration. Simply copy and paste.

```bash
cat > $HOME/validator.service << EOF 
# The eth2 validator service (part of systemd)
# file: /etc/systemd/system/validator.service 

[Unit]
Description     = eth2 validator service
Wants           = network-online.target beacon-chain.service
After           = network-online.target 

[Service]
User            = $(whoami)
WorkingDirectory= $(echo $HOME)/git/lodestar
ExecStart       = $(echo $HOME)/git/lodestar/lodestar validator --network mainnet --graffiti "${MY_GRAFFITI}"
Restart         = on-failure

[Install]
WantedBy	= multi-user.target
EOF
```

Move the unit file to `/etc/systemd/system`

```bash
sudo mv $HOME/validator.service /etc/systemd/system/validator.service
```

Update file permissions.

```bash
sudo chmod 644 /etc/systemd/system/validator.service
```

Run the following to enable auto-start at boot time and then start your validator.

```
sudo systemctl daemon-reload
sudo systemctl enable validator
sudo systemctl start validator
```

{% hint style="success" %}
Nice work. Your validator is now managed by the reliability and robustness of systemd. Below are some commands for using systemd.
{% endhint %}

:tools: **Some helpful systemd commands**

**🗄 Viewing and filtering logs**

```bash
#view and follow the log
journalctl --unit=validator -f
```

```bash
#view log since yesterday
journalctl --unit=validator --since=yesterday
```

```bash
#view log since today
journalctl --unit=validator --since=today
```

```bash
#view log between a date
journalctl --unit=validator --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```

:mag\_right: **View the status of the validator**

```
sudo systemctl status validator
```

:arrows\_counterclockwise: **Restarting the validator**

```
sudo systemctl reload-or-restart validator
```

:octagonal\_sign: **Stopping the validator**

```
sudo systemctl stop validator
```
{% endtab %}
{% endtabs %}
