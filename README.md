<h1 align=center>Gensyn Rl-Swarm Node Guide</h1>

## 💡 System Requirements

| Requirement                         | Details                                                     |
|-------------------------------------|-------------------------------------------------------------|
| **CPU Architecture**                | `arm64` or `amd64`                                          |
| **Recommended RAM**                 | 24 - 32 GB                                                       |
| **CUDA Devices**      | `RTX 3090`, `RTX 4070`, `RTX 4090`, `A100`, `H100`          |
| **Python Version**                  | Python >= 3.10            |

## 📊 Rent GPU
**Renting GPU is not necessarily needed, you can still run this node on VPS or on WSL.**
- Visit : [Clore.ai](https://clore.ai/)
- Sign Up using email address
- Click on `Account overview` button in the corner to deposit fund
- You can deposit using crypto currency CLORE from mexc, gate.io or usdt
- Now go to the `Marketplace` section, select a server with the required GPU, RAM, and region from the list below.
- Then, select General Purpose → Ubuntu Jupyter, cloreai/jupyter:ubuntu24.04-v2, and either save the configuration or set passwords for Jupyter (if you plan to use it) and SSH.
- Next, go to My Orders and find the information for the TCP connection, for example: n1.de.clorecloud.net:3333.

## ⚙️ Connect via SSH
- Now paste command like this on terminal to access your GPU server.
This is just an example — use your own TCP connection instead:
```bash 
ssh -p 3333 root@n1.de.clorecloud.net
```

## 🧩 Installation
1. **Install dependencies**
```bash
apt update && apt install -y sudo && \
sudo apt update && sudo apt install -y python3 python3-venv python3-pip curl wget screen git lsof nano unzip iproute2 && \
curl -sSL https://raw.githubusercontent.com/maloded/installers/main/node.sh | bash
```
2. **Install GPU drivers**
```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/sbsa/cuda-ubuntu2204.pin && \
sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600 && \
wget https://developer.download.nvidia.com/compute/cuda/12.6.0/local_installers/cuda-repo-ubuntu2204-12-6-local_12.6.0-560.28.03-1_arm64.deb && \
sudo dpkg -i cuda-repo-ubuntu2204-12-6-local_12.6.0-560.28.03-1_arm64.deb && \
sudo cp /var/cuda-repo-ubuntu2204-12-6-local/cuda-*-keyring.gpg /usr/share/keyrings/ && \
sudo apt-get update && \
sudo apt-get -y install cuda-toolkit-12-6
```
3. **Create a `screen` session**
```bash
screen -S gensyn
```
4. **Install Python dev headers, set up and activate virtual environment**
```bash
sudo apt-get update && \
sudo apt-get install -y python3.12-dev python3-venv && \
python3 -m venv .venv && \
source .venv/bin/activate
```
5. **Configure CUDA environment variables**
```bash
export PATH=/usr/local/cuda-12.6/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-12.6/lib64:$LD_LIBRARY_PATH
```
6. **Install Python packages**
```bash
pip install torch torchvision torchaudio hivemind colorlog
```
7. **Clone official `rl-swarm` repo**
```bash
git clone https://github.com/gensyn-ai/rl-swarm.git
```
8. **Upload existing swarm.pem to server**
> **If you already have a swarm.pem file and don’t need to create a new one, copy it to your server using scp. Replace the local path, server address, and port with your own values. Important: Run this command in a separate terminal window on your local machine, not on the server.**
```bash
# Variables (edit these according to your setup)
LOCAL_FILE_PATH="/path/to/your/swarm.pem"
REMOTE_USER="root"
REMOTE_HOST="your.server.address"
REMOTE_PORT=22
REMOTE_PATH="/root/rl-swarm/"

# Upload the file
scp -P $REMOTE_PORT $LOCAL_FILE_PATH $REMOTE_USER@$REMOTE_HOST:$REMOTE_PATH
```
9. **Run the swarm**
```bash
cd $HOME && rm -rf gensyn-setup && git clone https://github.com/maloded/gensyn-setup.git && chmod +x gensyn-setup/gensyn.sh && ./gensyn-setup/gensyn.sh
```
10. **SSH Login and Port Forwarding**
- To access services running on the remote server locally, you can create an SSH tunnel in your separate terminal window on your pc. This forwards a remote port to your local machine.
```bash
ssh -L <LOCAL_PORT>:localhost:<REMOTE_SERVICE_PORT> root@<SERVER_HOST> -p <SSH_PORT>
Example:
ssh -L 3003:localhost:3000 root@n1.de.clorecloud.net -p 3333
```
11. **Some questions**
> It will ask some questions, you should send response properly
- Would you like to push models you train in the RL swarm to the Hugging Face Hub? [y/N] >>> Press N to join testnet.
HuggingFace needs 2GB upload bandwidth for each model you train, you can press Y, and enter your access-token.
- Enter the name of the model you want to use in huggingface repo/name format, or press [Enter] to use the default model. >>> For default model, press Enter or choose one of these (More model parameters (B) need more vRAM):
- Gensyn/Qwen2.5-0.5B-Instruct
- Qwen/Qwen3-0.6B
- nvidia/AceInstruct-1.5B
- dnotitia/Smoothie-Qwen3-1.7B
- Gensyn/Qwen2.5-1.5B-Instruct
> During setup, you'll be asked if you'd like to participate in the AI Prediction Market.
- Example:
Would you like to participate in the AI Prediction Market? (Y/n)
You'll be entered into the prediction market by default, by pressing ENTER or answering Y to the Prediction Market prompt.
12. **Save Your Peer ID**
- Find the line in the logs that mentions your Peer ID.
- It usually starts with Qm and is followed by a long string of letters and numbers.
- Also note the three words shown in the logs (these are used to identify your node in the network).
- Save both the Peer ID (starting with Qm…) and the three words somewhere safe — you will need them later to connect or reference your node.
13. **Backup Your swarm.pem (if you just created your account and selected Step 2 during setup)**
```bash
scp -P $REMOTE_PORT $REMOTE_USER@$REMOTE_HOST:$REMOTE_PATH $LOCAL_PATH
Example: 
scp -P 3333 root@n1.de.clorecloud.net:/root/rl-swarm/swarm.pem /home/backup_dir
```
14. **Monitor Node Progress**
- Detach from Terminal Session, Ctrl + A, then D
- Visit [dashboard.gensyn.ai](https://dashboard.gensyn.ai/?application=RLSwarm) and login.
