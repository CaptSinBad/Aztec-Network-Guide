# Aztec-Network-Guide

Here’s a polished rephrasing for your guide:

---

**Step-by-Step Guide: Running a Sequencer Node on Aztec Testnet & Earning the Apprentice Role**

**Which Node Types Can Participate in the Testnet?**

* **Sequencer:** Proposes new blocks, validates blocks from other sequencers, and participates in governance votes.
* **Prover:** Generates zero-knowledge proofs to ensure the integrity of the rollup.

**Roles & Recognition**
Sequencer nodes are eligible for a specific role on Discord as recognition for their contributions. After setting up and fully syncing your Sequencer node, you can follow the “Get Role” process.

**Hardware Requirements**

| Node Type     | RAM     | CPU       | Disk                            |
| ------------- | ------- | --------- | ------------------------------- |
| **Sequencer** | 8–16 GB | 4–9 cores | 100+ GB SSD                     |
| **Prover**    | 128 GB  | 16 cores  | High-speed SSDs (~40x machines) |

> Note: Prover nodes require data-center level resources. For most users, including this guide, running a Prover is unnecessary.

---
VPS Users: can get started via a VPS with 4 cores CPU, 8GB RAM! [Purchase here](https://my.racknerd.com/aff.php?aff=13418)

Or for my brokies you can use google cloud free tier [here](https://cloud.google.com/free?hl=en) 
after signing up go to compute engine > VM instance > e2-standard-4 (4 vCPU, 2 core, 16 GB memory) > 
<img width="1302" height="581" alt="image" src="https://github.com/user-attachments/assets/58acee17-5d20-466a-bf73-4d3b2299b23b" />
NOTE THAT GOOGLE BLOCKS ANY FREE TIER VM THAT IS IS DOING CRYPTO RELATED STUFF ~3 days so you have to do asap and pray to get the role and then boom out

```
# Aztec Node Setup Guide

## 1. Install Dependencies

**Update packages:**
```bash
sudo apt-get update && sudo apt-get upgrade -y
````

**Install required packages:**

```
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

---
---
## 2. Install Docker

**Remove old versions (if any):**

```
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

**Add Docker’s official GPG key:**

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

**Set up the repository:**

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Install Docker:**

```
sudo apt update -y && sudo apt upgrade -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

**Test Docker:**

```
sudo docker run hello-world
```

**Enable and restart Docker:**

```
sudo systemctl enable docker
sudo systemctl restart docker
```

---

## 3. Install Aztec Tools

```
bash -i <(curl -s https://install.aztec.network)
```

**Add to PATH:**

```
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Restart your terminal** to apply changes.

**Verify installation:**

```
aztec
```

---

✅ Your system is now ready for running an **Aztec node**.


4. Obtain RPC URLs
Find a 3rd party that supports Sepolia RPC URL & Sepolia BEACON URL APIs.
Most of your usage is RPC URL. I recommend to use Alchemy for RPC URL & Use drpc for Beacon URL.
More details on RPC solutions:

**Get Your Own RPC by Running Geth & Prysm Nodes**
You can run your own local RPC nodes by following this guide: [geth-prysm-node](https://github.com/0xmoei/geth-prysm-node). You may need **600–1000 GB SSD**.

---
## 5. Generate Ethereum Keys

Create or import an **EVM wallet** (e.g. MetaMask, Rabby, etc.).  
⚠️ **Important:** Save your **private key** and **public address** securely — you’ll need them for validator configuration.

---

## 6. Get Sepolia ETH

Fund your Ethereum wallet with testnet ETH from a **Sepolia faucet**.  
- [Alchemy Sepolia Faucet](https://sepoliafaucet.com/)  
- [Infura Faucet](https://www.infura.io/faucet/sepolia)

---

## 7. Find Your Server’s Public IP

Run the following command and **save the IP** (you’ll use it for node config):  
```
curl ipv4.icanhazip.com
```

---

## 8. Enable Firewall & Open Ports

Configure UFW (Uncomplicated Firewall):

```
# SSH access
sudo ufw allow 22
sudo ufw allow ssh

# Sequencer ports
sudo ufw allow 40400
sudo ufw allow 8080

# Enable firewall
sudo ufw enable
```

---



## Configure the Sequencer

Setting up a sequencer involves configuring keys, environment variables, and Docker Compose. This includes creating the directory structure, defining private keys, and setting environment variables.

---

### 1. Create Directory Structure

```
mkdir -p aztec-sequencer/keys aztec-sequencer/data
cd aztec-sequencer
touch .env
```

---

### 2. Define Private Keys and Accounts

Create a `keystore.json` file inside `aztec-sequencer/keys`:

```
{
  "schemaVersion": 1,
  "validators": [
    {
      "attester": ["ETH_PRIVATE_KEY_0"],
      "publisher": ["ETH_PRIVATE_KEY_1"],
      "coinbase": "ETH_ADDRESS_2",
      "feeRecipient": "0x0000000000000000000000000000000000000000000000000000000000000000"
    }
  ]
}
```

🔑 **Replace the placeholders** with your actual keys and addresses:

* **attester** → private key for signing block proposals
* **publisher** → private key for posting proposals to L1 (must be funded with Sepolia ETH)
* **coinbase** → Ethereum address that receives L1 rewards and fees
* **feeRecipient** → Aztec address that receives unburnt transaction fees

⚠️ Make sure the `publisher` wallet is funded with at least **0.1 Sepolia ETH**.

---

### 3. Configure Environment Variables

Add the following to your `.env` file:

```
DATA_DIRECTORY=./data
KEY_STORE_DIRECTORY=./keys
LOG_LEVEL=info
ETHEREUM_HOSTS=<your L1 execution RPC endpoint>
L1_CONSENSUS_HOST_URLS=<your L1 consensus RPC endpoint>
P2P_IP=<your external IP address>
P2P_PORT=40400
AZTEC_PORT=8080
```

---

  
🔑 Replace the placeholders with your actual keys and addresses:

The keystore defines keys and addresses for sequencer operation:

attester: Private key for signing block proposals and attestations. The corresponding Ethereum address identifies the sequencer.
publisher: Private key for sending block proposals to L1. Defaults to attester key if not set.
coinbase: Ethereum address receiving L1 rewards and fees. Defaults to attester address if not set.
feeRecipient: Aztec address receiving unburnt transaction fees from blocks.
NOTE: YOU CAN CHANGE AND SHOULD THE feeRecipient ADDRESS AFTER SETTING UP THE NODE TO YOUR OWN AZTEC ADDRESS BY CREATING AN AZTEC ADDRESS
USING:
```
aztec-wallet create-account -h
```

---


# Aztec Sequencer: Auto-Update, Deployment, and Verification

## Enable Auto-Update and Auto-Restart

The sequencer's auto-update functionality is **critical for network coordination**. This background module enables:

- Configuration updates across all nodes
- Automated image updates via controlled shutdowns
- Rapid hot-fix deployment
- Coordinated resets after governance upgrades

> **Important:**  
> Do **NOT** set `AUTO_UPDATE_URL` or `AUTO_UPDATE` environment variables. These must use their default values for proper operation.

Since Docker Compose doesn't respect pull policies on container restarts, install [Watchtower](https://containrrr.dev/watchtower/) for automatic updates:

```sh
docker run -d \
  --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower
```

---

## Deploy with Docker Compose

Create a `docker-compose.yml` file in your `aztec-sequencer` directory:

```yaml
services:
  aztec-sequencer:
    image: "aztecprotocol/aztec:latest"
    container_name: "aztec-sequencer"
    ports:
      - ${AZTEC_PORT}:${AZTEC_PORT}
      - ${P2P_PORT}:${P2P_PORT}
      - ${P2P_PORT}:${P2P_PORT}/udp
    volumes:
      - ${DATA_DIRECTORY}:/var/lib/data
      - ${KEY_STORE_DIRECTORY}:/var/lib/keystore
    environment:
      KEY_STORE_DIRECTORY: /var/lib/keystore
      DATA_DIRECTORY: /var/lib/data
      LOG_LEVEL: ${LOG_LEVEL}
      ETHEREUM_HOSTS: ${ETHEREUM_HOSTS}
      L1_CONSENSUS_HOST_URLS: ${L1_CONSENSUS_HOST_URLS}
      P2P_IP: ${P2P_IP}
      P2P_PORT: ${P2P_PORT}
      AZTEC_PORT: ${AZTEC_PORT}
    entrypoint: >-
      node
      --no-warnings
      /usr/src/yarn-project/aztec/dest/bin/index.js
      start
      --node
      --archiver
      --sequencer
      --network testnet
    networks:
      - aztec
    restart: always

networks:
  aztec:
    name: aztec
```

> **Note:**  
> This configuration includes only essential settings. The `--network testnet` flag applies network-specific defaults.  
> See the [CLI reference](https://docs.aztec.network/the_aztec_network/reference/cli_reference) for all available options.

---

## Start the Sequencer

```sh
docker compose up -d
```

---

## Verification

To verify your sequencer is running correctly:

### 1. Check the current sync status (this may take a few minutes):

```sh
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number"
```

Compare the output with block explorers like [Aztec Scan](https://aztecscan.xyz/) or [Aztec Explorer](https://aztecexplorer.xyz/).

---

### 2. View sequencer logs

```sh
docker compose logs -f aztec-sequencer
```

---

### 3. Check node status

```sh
curl http://localhost:8080/status
```

---

**Source:**  
[Aztec Docs – How to Run a Sequencer Node](https://docs.aztec.network/the_aztec_network/guides/run_nodes/how_to_run_sequencer)

---

