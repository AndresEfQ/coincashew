---
description: >-
  Become a validator and help secure eth2, a proof-of-stake blockchain. Anyone
  with 32 ETH can join.
---

# Guide: How to stake on ETH2 Mainnet with Teku on Ubuntu

{% hint style="danger" %}
**Nov 24 2020 Update**: The [new mainnet guide is located here](guide-or-how-to-setup-a-validator-on-eth2-mainnet/).&#x20;

Instructions below are now deprecated and for reference only.
{% endhint %}











{% hint style="info" %}
[PegaSys Teku](https://pegasys.tech/teku/) (formerly known as Artemis) is a Java-based Ethereum 2.0 client designed & built to meet institutional needs and security requirements. PegaSys is an arm of [ConsenSys](https://consensys.net) dedicated to building enterprise-ready clients and tools for interacting with the core Ethereum platform. Teku is Apache 2 licensed and written in Java, a language notable for its materity & ubiquity.
{% endhint %}

## :checkered\_flag: 0. Prerequisites

### :woman\_technologist: Skills for operating a eth2 validator and beacon node

As a validator for eth2, you will typically have the following abilities:

* operational knowledge of how to set up, run and maintain a eth2 beacon node and validator continuously
* a commitment to maintain your validator 24/7/365
* basic operating system skills
* have learned the essentials by watching ['Intro to Eth2 & Staking for Beginners' by Superphiz](https://www.youtube.com/watch?v=tpkpW031RCI)
* have passed or is actively enrolled in the [Eth2 Study Master course](https://ethereumstudymaster.com)
* and have read the [8 Things Every Eth2 validator should know.](https://medium.com/chainsafe-systems/8-things-every-eth2-validator-should-know-before-staking-94df41701487)

### ****:reminder\_ribbon: **Minimum Setup Requirements**

* **Operating system:** 64-bit Linux (i.e. Ubuntu 20.04 LTS)
* **Processor:** Dual core CPU, Intel Core i5–760 or AMD FX-8100 or better
* **Memory:** 8GB RAM
* **Storage:** 20GB SSD
* **Internet:** Broadband internet connection with speeds at least 1 Mbps.
* **Power:** Reliable electrical power.
* **ETH balance:** at least 32 ETH and some ETH for deposit transaction fees
* **Wallet**: Metamask installed

### :woman\_lifting\_weights: Recommended Hardware Setup

* **Operating system:** 64-bit Linux (i.e. Ubuntu 20.04 LTS)
* **Processor:** Quad core CPU, Intel Core i7–4770 or AMD FX-8310 or better
* **Memory:** 16 GB RAM or more
* **Storage:** 1TB SSD or more&#x20;
* **Internet:** Broadband internet connections with speeds at least 10 Mbps
* **Power:** Reliable electrical power with uninterruptible power supply (UPS)
* **ETH balance:** at least 32 ETH and some ETH for deposit transaction fees
* **Wallet**: Metamask installed

{% hint style="warning" %}
:sparkles: **Pro Validator Tip**: Highly recommend you begin with a brand new instance of an OS, VM, and/or machine. Avoid headaches by NOT reusing testnet keys, wallets, or databases for your mainnet validator.
{% endhint %}

### :unlock: Recommended eth2 validator Security Best Practices

If you need ideas or a reminder on how to secure your validator, refer to

{% content-ref url="broken-reference" %}
[Broken link](broken-reference)
{% endcontent-ref %}

### :tools: Setup Ubuntu

If you need to install Ubuntu, refer to

{% content-ref url="../overview-xtz/guide-how-to-setup-a-baker/install-ubuntu.md" %}
[install-ubuntu.md](../overview-xtz/guide-how-to-setup-a-baker/install-ubuntu.md)
{% endcontent-ref %}

### :performing\_arts: Setup Metamask

If you need to install Metamask, refer to

{% content-ref url="../../wallets/browser-wallets/metamask-ethereum.md" %}
[metamask-ethereum.md](../../wallets/browser-wallets/metamask-ethereum.md)
{% endcontent-ref %}

## :seedling: 1. Buy/exchange or consolidate ETH

{% hint style="info" %}
Every 32 ETH you own allows you to make 1 validator. You can run thousands of validators with your beacon node.
{% endhint %}

Your ETH (or multiples of 32 ETH) should be consolidated into a single address accessible with Metamask.

If you need to buy/exchange or top up your ETH to a multiple of 32, check out:

{% content-ref url="guide-how-to-buy-eth.md" %}
[guide-how-to-buy-eth.md](guide-how-to-buy-eth.md)
{% endcontent-ref %}

## :woman\_technologist:2. Signup to be a validator at the Launchpad

1. Install dependencies, the ethereum foundation deposit tool and generate your two sets of key pairs.

{% hint style="info" %}
Each validator will have two sets of key pairs. A **signing key** and a **withdrawal key.** These keys are derived from a single mnemonic phrase. [Learn more about keys.](https://blog.ethereum.org/2020/05/21/keys/)
{% endhint %}

You have the choice of downloading the pre-built [ethereum foundation deposit tool](https://github.com/ethereum/eth2.0-deposit-cli) or building it from source.

{% tabs %}
{% tab title="Pre-built eth2deposit-cli" %}
Download eth2deposit-cli.

```bash
cd $HOME
wget https://github.com/ethereum/eth2.0-deposit-cli/releases/download/v1.0.0/eth2deposit-cli-9310de0-linux-amd64.tar.gz
```

Verify the SHA256 Checksum matches the checksum on the [releases page](https://github.com/ethereum/eth2.0-deposit-cli/releases/tag/v1.0.0).

```bash
sha256sum eth2deposit-cli-9310de0-linux-amd64.tar.gz 
# SHA256 should be
# b09da136895a7f77a4b430924ea2ae5827fa47b2bf444c4ea6fcfac5b04b8c8a
```

Extract the archive.

```
tar -xvf eth2deposit-cli-9310de0-linux-amd64.tar.gz
cd eth2deposit-cli-9310de0-linux-amd64
```

Make a new mnemonic.

```
./deposit new-mnemonic --chain mainnet
```
{% endtab %}

{% tab title="Build from source code" %}
Install dependencies.

```
sudo apt update
sudo apt install python3-pip git -y
```

Download source code and install.

```
mkdir ~/git
cd ~/git
git clone https://github.com/ethereum/eth2.0-deposit-cli.git
cd eth2.0-deposit-cli
sudo ./deposit.sh install
```

Make a new mnemonic.

```
./deposit.sh new-mnemonic --chain mainnet
```
{% endtab %}

{% tab title="Advanced - Most Secure" %}
{% hint style="warning" %}
:fire:**\[ Optional ] Pro Security Tip**: Run the eth2deposit-cli tool and generate your **mnemonic seed** for your validator keys on an **air-gapped offline machine**.&#x20;

You can copy via USB key the pre-built eth2deposit-cli binaries from an online machine to an air-gapped offline machine.

* Protects against key-logging attacks, malware/virus based attacks and other firewall or security exploits.&#x20;
* Physically isolated from the rest of your network.&#x20;
* Must not have a network connection, wired or wireless.&#x20;
* Is not a VM on a machine with a network connection.
* Learn more about [air-gapping at wikipedia](https://en.wikipedia.org/wiki/Air\_gap\_\(networking\)).
{% endhint %}
{% endtab %}
{% endtabs %}

2\. Follow the prompts and pick a password. Write down your mnemonic and keep this safe and **offline**.

3\. Follow the steps at [https://launchpad.ethereum.org/](https://launchpad.ethereum.org) while skipping over the steps you already just completed. Study the eth2 phase 0 overview material. Understanding eth2 is the key to success!

4\. Back on the launchpad website, upload your`deposit_data-#########.json` found in the `validator_keys` directory.

5\. Connect to the launchpad with your Metamask wallet, review and accept terms.

6\. Confirm the transaction(s). There's one deposit transaction of 32 ETH for each validator.

{% hint style="info" %}
Your transaction is sending and depositing your ETH to the [official ETH2 deposit contract address. ](https://blog.ethereum.org/2020/11/04/eth2-quick-update-no-19/)

**Check**, _double-check_, _**triple-check**_ that the official Eth2 deposit contract address is correct.[`0x00000000219ab540356cBB839Cbe05303d7705Fa`](https://etherscan.io/address/0x00000000219ab540356cbb839cbe05303d7705fa)&#x20;
{% endhint %}

{% hint style="danger" %}
Be sure to write down or record your mnemonic seed **offline**. _Not email. Not cloud._

Make **offline backups**, such as to a USB key, of your **`validator_keys`**` ``` directory.
{% endhint %}

## :flying\_saucer:3. Install a ETH1 node

{% hint style="info" %}
Ethereum 2.0 requires a connection to Ethereum 1.0 in order to monitor for 32 ETH validator deposits. Hosting your own Ethereum 1.0 node is the best way to maximize decentralization and minimize dependency on third parties such as Infura.
{% endhint %}

{% hint style="warning" %}
The subsequent steps assume you have completed the [best practices security guide](broken-reference).
{% endhint %}

Your choice of either [**OpenEthereum**](https://www.parity.io/ethereum/)**,** [**Geth**](https://geth.ethereum.org)**,** [**Besu**](https://besu.hyperledger.org) **or** [**Nethermind**](https://www.nethermind.io)**.**

{% tabs %}
{% tab title="OpenEthereum (Parity)" %}
#### &#x20;:robot: Install and run OpenEthereum.

```
mkdir ~/openethereum && cd ~/openethereum
wget https://github.com/openethereum/openethereum/releases/download/v3.0.1/openethereum-linux-v3.0.1.zip
unzip openethereum*.zip
chmod +x openethereum
rm openethereum*.zip
```

​ :gear: **Setup and configure systemd**

Run the following to create a **unit file** to define your `eth1.service` configuration.

```bash
cat > $HOME/eth1.service << EOF 
[Unit]
Description     = openethereum eth1 service
Wants           = network-online.target
After           = network-online.target 

[Service]
User            = $(whoami)
WorkingDirectory= /home/$(whoami)/openethereum
ExecStart       = /home/$(whoami)/openethereum/openethereum --chain foundation
Restart         = on-failure

[Install]
WantedBy	= multi-user.target
EOF
```

Move the unit file to `/etc/systemd/system` and give it permissions.

```bash
sudo mv $HOME/eth1.service /etc/systemd/system/eth1.service
```

```bash
sudo chmod 644 /etc/systemd/system/eth1.service
```

Run the following to enable auto-start at boot time.

```
sudo systemctl daemon-reload
sudo systemctl enable eth1
```

#### :chains: Start OpenEthereum on mainnet.

```
sudo systemctl start eth1
```
{% endtab %}

{% tab title="Geth" %}
#### :dna: Install from the repository.

```
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update -y
sudo apt-get install ethereum -y
```

:gear: **Setup and configure systemd**

Run the following to create a **unit file** to define your `eth1.service` configuration.

```bash
cat > $HOME/eth1.service << EOF 
[Unit]
Description     = geth eth1 service
Wants           = network-online.target
After           = network-online.target 

[Service]
User            = $(whoami)
ExecStart       = /usr/bin/geth --http
Restart         = on-failure

[Install]
WantedBy	= multi-user.target
EOF
```

Move the unit file to `/etc/systemd/system` and give it permissions.

```bash
sudo mv $HOME/eth1.service /etc/systemd/system/eth1.service
sudo chmod 644 /etc/systemd/system/eth1.service
```

Run the following to enable auto-start at boot time.

```
sudo systemctl daemon-reload
sudo systemctl enable eth1
```

#### :chains: Start geth on mainnet.

```
sudo systemctl start eth1
```
{% endtab %}

{% tab title="Besu" %}
#### :dna: Install java dependency.

```
sudo apt install openjdk-11-jdk
```

#### :last\_quarter\_moon\_with\_face: Download and unzip Besu.

```
cd
wget -O besu.tar.gz https://bintray.com/hyperledger-org/besu-repo/download_file?file_path=besu-1.5.0.tar.gz
tar -xvf besu.tar.gz
rm besu.tar.gz
mv besu-1.5.0 besu
```

:gear: **Setup and configure systemd**

Run the following to create a **unit file** to define your `eth1.service` configuration.

```bash
cat > $HOME/eth1.service << EOF 
[Unit]
Description     = openethereum eth1 service
Wants           = network-online.target
After           = network-online.target 

[Service]
User            = $(whoami)
WorkingDirectory= /home/$(whoami)/besu/bin
ExecStart       = /home/$(whoami)/besu/bin/besu --data-path="$HOME/.ethereum_besu"
Restart         = on-failure

[Install]
WantedBy	= multi-user.target
EOF
```

Move the unit file to `/etc/systemd/system` and give it permissions.

```bash
sudo mv $HOME/eth1.service /etc/systemd/system/eth1.service
sudo chmod 644 /etc/systemd/system/eth1.service
```

Run the following to enable auto-start at boot time.

```
sudo systemctl daemon-reload
sudo systemctl enable eth1
```

#### :chains: Start besu on mainnet.

```
sudo systemctl start eth1
```
{% endtab %}

{% tab title="Nethermind" %}
#### :gear: Install dependencies.

```
sudo apt-get update && sudo apt-get install libsnappy-dev libc6-dev libc6 unzip -y
```

#### :last\_quarter\_moon\_with\_face: Download and unzip Nethermind.

```
mkdir ~/nethermind && cd ~/nethermind
wget -O nethermind.zip https://nethdev.blob.core.windows.net/builds/nethermind-linux-amd64-1.8.77-9d3a58a.zip
unzip nethermind.zip
rm nethermind.zip
```

#### :flying\_saucer: Launch Nethermind.

```
./Nethermind.Launcher
```

* Select `Ethereum Node`
* Select `Ethereum (mainnet)` then select `Fast sync`&#x20;
* Yes to enable web3 / JSON RPC
* Accept default IP
* Skip ethstats registration
{% endtab %}

{% tab title="Minimum Hardware Setup" %}
:construction: **Untested - TBD - Work in progress** :construction:&#x20;

Use a third party by signing up for an API access key at [https://infura.io/](https://infura.io)
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Syncing the eth1 node could take up to 24 hour.
{% endhint %}

{% hint style="success" %}
Your eth1 node is fully sync'd when these events occur.

* **`OpenEthereum:`**` ``Imported #<block number>`
* **`Geth:`**` ``Imported new chain segment`
* **`Besu:`**` ``Imported #<block number>`
* **`Nethermind:`**` ``No longer syncing Old Headers`
{% endhint %}

#### :tools: Helpful eth1.service commands

​​ :notepad\_spiral: **To view and follow eth1 logs**

```
journalctl -u eth1 -f
```

:notepad\_spiral: **To stop eth1 service**

```
sudo systemctl stop eth1
```

{% hint style="danger" %}
:octagonal\_sign: **Before continuing the rest of this guide, we recommend you wait until closer to Dec 1st as the Teku code is rapidly preparing for mainnet.** :construction:&#x20;
{% endhint %}

## :bulb: 4. Build Teku from source

Install git.

```
sudo apt-get install git -y
```

Install Java 11.

{% tabs %}
{% tab title="Ubuntu 20.x" %}
```
sudo apt update
sudo apt install openjdk-11-jdk
```
{% endtab %}

{% tab title="Ubuntu 18.x" %}
```
sudo add-apt-repository ppa:linuxuprising/java
sudo apt update
sudo apt install oracle-java11-set-default
```
{% endtab %}
{% endtabs %}

Install and build Teku.

```bash
cd ~/git
git clone https://github.com/PegaSysEng/teku.git
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

Copy the teku binary files to `/usr/local/teku`

```bash
sudo cp -r $HOME/git/teku/build/install/teku/. /usr/bin/teku
```

## :fire: 5. Configure port forwarding and/or firewall

Specific to your networking setup or cloud provider settings, [ensure your validator's firewall ports are open and reachable.](broken-reference)

* **Teku beacon chain node** will use port 9001 for tcp and udp
* **eth1** node requires port 30303 for tcp and udp

{% hint style="info" %}
****:sparkles: **Port Forwarding Tip:** You'll need to forward and open ports to your validator. Verify it's working with [https://www.yougetsignal.com/tools/open-ports/](https://www.yougetsignal.com/tools/open-ports/) or [https://canyouseeme.org/](https://canyouseeme.org) .
{% endhint %}

## :snowboarder: 6. Start the beacon chain and validator

{% hint style="info" %}
Teku combines both the beacon chain and validator into one process.
{% endhint %}

{% hint style="warning" %}
If you participated in any of the prior test nets, you need to clear the database.

```bash
rm -rf $HOME/.local/share/teku/data
```
{% endhint %}

Setup a directory structure for Teku.

```bash
sudo mkdir -p /var/lib/teku
sudo mkdir -p /etc/teku
```

&#x20;Copy your `validator_files` directory to the data directory we created above.

{% tabs %}
{% tab title="Pre-built eth2deposit-cli" %}
```bash
sudo cp -r $HOME/eth2deposit-cli-9310de0-linux-amd64/validator_keys /var/lib/teku
```
{% endtab %}

{% tab title="Built from source code" %}
```bash
sudo cp -r $HOME/git/eth2.0-deposit-cli/validator_keys /var/lib/teku
```
{% endtab %}
{% endtabs %}

Store your validator's password in a file.

```bash
echo "my_password_goes_here" > $HOME/validators-password.txt
sudo mv $HOME/validators-password.txt /etc/teku/validators-password.txt
```

Generate your Teku Config file.

```bash
cat > $HOME/teku.yaml << EOF
# network
network: "mainnet"

# p2p
p2p-enabled: true
p2p-port: 9001

# validators
validator-keys: "/var/lib/teku/validator_keys:/etc/teku/validator_keys"
validators-graffiti: "Teku validator & CoinCashew.com"

# Eth 1
eth1-endpoint: "http://localhost:8545"

# metrics
metrics-enabled: true
metrics-categories: ["BEACON","LIBP2P","NETWORK"]
metrics-port: 8008

# database
data-path: "$(echo $HOME)/tekudata"
data-storage-mode: "archive"

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

{% hint style="info" %}
When specifying directories for your validator-keys, Teku expects to find identically named keystore and password files.&#x20;

For example `keystore-m_12221_3600_1_0_0-11222333.json` and `keystore-m_12221_3600_1_0_0-11222333.txt`
{% endhint %}

Create a corresponding password file for every one of your validators.

```bash
for f in /var/lib/teku/validator_keys/keystore*.json; do cp /etc/teku/validators-password.txt /var/lib/teku/validator_keys/$(basename $f .json).txt; doneYour choice of running a beacon chain and validator manually from command line or automatically with systemd.
```

{% tabs %}
{% tab title="Systemd - Automated" %}
#### 🍰 Benefits of using systemd for your beacon chain and validator <a href="#benefits-of-using-systemd-for-your-stake-pool" id="benefits-of-using-systemd-for-your-stake-pool"></a>

1. Auto-start your beacon chain when the computer reboots due to maintenance, power outage, etc.
2. Automatically restart crashed beacon chain processes.
3. Maximize your beacon chain up-time and performance.

#### :tools: Setup Instructions

Run the following to create a **unit file** to define your`beacon-chain.service` configuration.

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
WorkingDirectory= /usr/local/teku/bin
ExecStart       = /usr/local/teku/bin/teku -c /etc/teku/teku.yaml
Restart         = on-failure

[Install]
WantedBy	= multi-user.target
EOF
```

Move the unit file to `/etc/systemd/system` and give it permissions.

```bash
sudo mv $HOME/beacon-chain.service /etc/systemd/system/beacon-chain.service
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

### :tools: Some helpful systemd commands

#### :white\_check\_mark: Check whether the beacon chain is active

```
sudo systemctl is-active beacon-chain
```

#### :mag\_right: View the status of the beacon chain

```
sudo systemctl status beacon-chain
```

#### :arrows\_counterclockwise: Restarting the beacon chain

```
sudo systemctl reload-or-restart beacon-chain
```

#### :octagonal\_sign: Stopping the beacon chain

```
sudo systemctl stop beacon-chain
```

#### 🗄 Viewing and filtering logs

```bash
journalctl --unit=beacon-chain --since=yesterday
journalctl --unit=beacon-chain --since=today
journalctl --unit=beacon-chain --since='2020-12-01 00:00:00' --until='2020-12-02 12:00:00'
```
{% endtab %}

{% tab title="CLI - Manual" %}
In a new terminal, start the beacon chain.

```bash
/usr/local/teku/bin/teku -c /etc/teku/teku.yaml
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
**WARNING**: DO NOT USE THE ORIGINAL KEYSTORES TO VALIDATE WITH ANOTHER CLIENT, OR YOU WILL GET SLASHED.
{% endhint %}

{% hint style="info" %}
**Validator client** - Responsible for producing new blocks and attestations in the beacon chain and shard chains.

**Beacon chain client** - Responsible for managing the state of the beacon chain, validator shuffling, and more.
{% endhint %}

{% hint style="success" %}
Congratulations. Once your beacon-chain is sync'd, validator up and running, you just wait for activation. This process takes up to 24 hours. When you're assigned, your validator will begin creating and voting on blocks while earning ETH staking rewards.&#x20;

Use [beaconcha.in](https://beaconcha.in) and [register an account](https://beaconcha.in/register) to create alerts and track your validator's performance.
{% endhint %}

## :clock3: 7. Time Synchronization

{% hint style="info" %}
Because beacon chain relies on accurate times to perform attestations and produce blocks, your computer's time must be accurate to real NTP or NTS time within 0.5 seconds.
{% endhint %}

Setup **Chrony** with the following guide.

{% content-ref url="broken-reference" %}
[Broken link](broken-reference)
{% endcontent-ref %}

{% hint style="info" %}
chrony is an implementation of the Network Time Protocol and helps to keep your computer's time synchronized with NTP.
{% endhint %}

## :mag\_right: 8. Monitoring your validator with Grafana and Prometheus

Prometheus is a monitoring platform that collects metrics from monitored targets by scraping metrics HTTP endpoints on these targets. [Official documentation is available here.](https://prometheus.io/docs/introduction/overview/) Grafana is a dashboard used to visualize the collected data.

### &#x20;:hatching\_chick: 8.1 Installation

Install prometheus and prometheus node exporter.

```
sudo apt-get install -y prometheus prometheus-node-exporter 
```

Install grafana.

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" > grafana.list
sudo mv grafana.list /etc/apt/sources.list.d/grafana.list
sudo apt-get update && sudo apt-get install -y grafana
```

Enable services so they start automatically.

```bash
sudo systemctl enable grafana-server.service
sudo systemctl enable prometheus.service
sudo systemctl enable prometheus-node-exporter.service
```

Update **prometheus.yml** located in `/etc/prometheus/prometheus.yml`

```bash
cat > $HOME/prometheus.yml << EOF
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'codelab-monitor'

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
   - job_name: 'node_exporter'
     static_configs:
       - targets: ['localhost:9100']
   - job_name: 'nodes'
     metrics_path: /metrics    
     static_configs:
       - targets: ['localhost:8008']
EOF
sudo mv $HOME/prometheus.yml /etc/prometheus/prometheus.yml
```

Finally, restart the services.

```bash
sudo systemctl restart grafana-server.service
sudo systemctl restart prometheus.service
sudo systemctl restart prometheus-node-exporter.service
```

Verify that the services are running properly:

```
sudo systemctl status grafana-server.service prometheus.service prometheus-node-exporter.service
```

{% hint style="info" %}
****:bulb: **Reminder**: Ensure port 3000 is open on the firewall and/or port forwarded if you intend to view monitoring info from a different machine.
{% endhint %}

### :signal\_strength:8.2 Setting up Grafana Dashboards&#x20;

1. Open [http://localhost:3000](http://localhost:3000) or http://\<your validator's ip address>:3000 in your local browser.
2. Login with **admin** / **admin**
3. Change password
4. Click the **configuration gear** icon, then **Add data Source**
5. Select **Prometheus**
6. Set **Name** to **"Prometheus**"
7. Set **URL** to **http://localhost:9090**
8. Click **Save & Test**
9. **Download and save** this [**json file.**](https://grafana.com/api/dashboards/12523/revisions/2/download)****
10. Click **Create +** icon > **Import**
11. Add dashboard by **Upload JSON file**
12. Click the **Import** button.

![](../../.gitbook/assets/graf-teku-dash.png)

### :warning: 8.3 Setup Alert Notifications

{% hint style="info" %}
Setup alerts to get notified if your validators go offline.
{% endhint %}

Get notified of problems with your validators. Choose between email, telegram, discord or slack.

{% tabs %}
{% tab title="Email Notifications" %}
1. Visit [https://beaconcha.in/](https://beaconcha.in)
2. Sign Up **** for an **account**
3. Verify your **email**
4. Search for your **validator's public address**
5. Add validators to your watchlist by clicking the **bookmark symbol**.
{% endtab %}

{% tab title="Telegram Notifications" %}
1. On the menu of Grafana, select **Notification channels** under the bell icon. <img src="../../.gitbook/assets/gra-noti.png" alt="" data-size="original">&#x20;
2. Click on **Add channel**.
3. Give the notification channel a **name**.
4. Select **Telegram** from the Type list.
5. To complete the **Telegram API settings**, a Telegram channel and bot are required. For instructions on setting up a bot with `@Botfather`, see [this section](https://core.telegram.org/bots#6-botfather) of the Telegram documentation.
6. Once completed, invite the bot to the newly created channel.
{% endtab %}

{% tab title="Discord Notifications" %}
1. On the menu of Grafana, select **Notification channels** under the bell icon. <img src="../../.gitbook/assets/gra-noti.png" alt="" data-size="original">&#x20;
2. Click on **Add channel**.
3. Add a **name** to the notification channel.
4. Select **Discord** from the Type list.
5. To complete the set up, a Discord server (and a text channel available) as well as a Webhook URL are required. For instructions on setting up a Discord's Webhooks, see [this section](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks) of their documentation.
6. Enter the Webhook **URL** in the Discord notification settings panel.
7. Click **Send Test**, which will push a confirmation message to the Discord channel.
{% endtab %}

{% tab title="Slack Notifications" %}
1. On the menu of Grafana, select **Notification channels** under the bell icon. <img src="../../.gitbook/assets/gra-noti.png" alt="" data-size="original">&#x20;
2. Click on **Add channel**.
3. Add a **name** to the notification channel.
4. Select **Slack** from the Type list.
5. For instructions on setting up a Slack's Incoming Webhooks, see [this section](https://api.slack.com/messaging/webhooks) of their documentation.
6. Enter the Slack Incoming Webhook URL in the **URL** field.
7. Click **Send Test**, which will push a confirmation message to the Slack channel.
{% endtab %}
{% endtabs %}

{% hint style="success" %}
:tada: Congrats on setting up your validator! You're good to go on eth2.0.

Did you find our guide useful? Let us know with a tip and we'll keep updating it.&#x20;

Use [cointr.ee to find our donation ](https://cointr.ee/coincashew)addresses. :pray:&#x20;

Any feedback and all pull requests much appreciated. :blush:&#x20;

Hang out and chat with fellow stakers on telegram @ [https://t.me/coincashew](https://t.me/coincashew) :first\_quarter\_moon\_with\_face:&#x20;
{% endhint %}

## :man\_mage: 9. Updating Teku

```bash
cd ~/git/teku
git pull
./gradlew distTar installDist
```

Restart beacon chain and validator as per normal operating procedures.

{% tabs %}
{% tab title="Systemd - Automated" %}
```bash
sudo systemctl stop beacon-chain
sudo cp -r $HOME/git/teku/build/install/teku/. /usr/bin/teku
sudo systemctl reload-or-restart beacon-chain
```
{% endtab %}

{% tab title="CLI - Manual" %}
```bash
killall teku
sudo cp -r $HOME/git/teku/build/install/teku/. /usr/bin/teku
/usr/local/teku/bin/teku -c /etc/teku/teku.yaml
```
{% endtab %}
{% endtabs %}

## :jigsaw: 10. Reference Material

Appreciate the hard work done by the fine folks at the following links which served as a foundation for creating this guide.

{% embed url="https://discord.gg/7hPv2T6" %}

{% embed url="https://launchpad.ethereum.org/" %}

{% embed url="https://pegasys.tech/teku-ethereum-2-for-enterprise/" %}

{% embed url="https://docs.teku.pegasys.tech/en/latest/HowTo/Get-Started/Build-From-Source/" %}

## &#x20;:tada: 11. Bonus Links

### :chestnut: CoinCashew Guides for other ETH2 Clients

{% content-ref url="guide-how-to-stake-on-eth2-with-lighthouse.md" %}
[guide-how-to-stake-on-eth2-with-lighthouse.md](guide-how-to-stake-on-eth2-with-lighthouse.md)
{% endcontent-ref %}

{% content-ref url="guide-how-to-stake-on-eth2.md" %}
[guide-how-to-stake-on-eth2.md](guide-how-to-stake-on-eth2.md)
{% endcontent-ref %}

{% content-ref url="guide-how-to-stake-on-eth2-with-nimbus.md" %}
[guide-how-to-stake-on-eth2-with-nimbus.md](guide-how-to-stake-on-eth2-with-nimbus.md)
{% endcontent-ref %}

{% content-ref url="guide-how-to-stake-on-eth2-with-lodestar.md" %}
[guide-how-to-stake-on-eth2-with-lodestar.md](guide-how-to-stake-on-eth2-with-lodestar.md)
{% endcontent-ref %}

### :bricks: ETH2 Block Explorers

{% embed url="https://beaconcha.in" %}

{% embed url="https://beaconscan.com" %}

### :notepad\_spiral: Latest Eth2 Info

{% embed url="https://www.reddit.com/r/ethstaker" %}

{% embed url="https://blog.ethereum.org" %}

{% embed url="http://invite.gg/ethstaker" %}

{% embed url="https://hackmd.io/@benjaminion/eth2_news/" %}

## :fire: 12. Additional Useful Tips

### :octagonal\_sign: 12.1 Voluntary exit a validator

{% hint style="info" %}
Use this command to signal your intentions to stop validating with your validator. This means you no longer want to stake with your validator and want to turn off your node.

* Voluntary exiting takes a minimum of 2048 epochs (or \~9days). There is a queue to exit and a delay before your validator is finally exited.
* Once a validator is exited in phase 0, this is non-reversible and you can no longer restart validating again.&#x20;
* Your funds will not be available for withdrawal until phase 1.5 or later.&#x20;
* After your validator leaves the exit queue and is truely exited, it is safe to turn off your beacon node and validator.
{% endhint %}

```bash
#TO BE DETERMINED
```

### :closed\_lock\_with\_key: 12.2 Verify your mnemonic phrase

Using the eth2deposit-cli tool, ensure you can regenerate the same eth2 key pairs by restoring your `validator_keys`&#x20;

```bash
./deposit existing-mnemonic --chain mainnet
```

{% hint style="info" %}
When the **pubkey** is identical, this means your **keystore file** you correctly verified your mnemonic phrase. Other fields will be different because of salting.
{% endhint %}

### :robot: 12.3 Add additional validators

Using the eth2deposit-cli tool, you can add more validators by creating a new deposit data file and `validator_keys`

For example, in case we originally created 3 validators but now wish to add 5 more validators, we could use the following command.

```bash
./deposit existing-mnemonic --validator_start_index 3 --num_validators 5 --chain mainnet
```

Complete the steps of uploading the `deposit_data-#########.json` to the launch pad site.
