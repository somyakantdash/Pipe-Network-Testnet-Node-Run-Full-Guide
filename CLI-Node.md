# CLI Node Run Full Guide (PC and VPS for Both)

### Offical Docs by Pipe Network - https://docs.pipe.network/nodes/testnet

----

## 🧰 Prerequisites (Only For Devnet Node Run Users)
	
### Stop Old Node
If you have running a devnet node previously, you need to do these steps first.

### 1. Backup `Node Info File`
You will need to backup old `node_info.json` file which you can find in pipe folder in your `root` directory.

Save the file
```
nano ~/node_info.json
```

### 2. Stop old node
```
sudo systemctl stop pipe
sudo systemctl disable pipe
sudo systemctl daemon-reload
```

---

1️⃣ Dependencies for WINDOWS & LINUX & VPS
```
sudo apt-get update && sudo apt-get upgrade -y
```
```
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y
```
```
sudo apt install -y libssl-dev ca-certificates
```

2️⃣ Enable Firewall & Open Ports
```
sudo ufw allow ssh
sudo ufw enable

sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw reload
```

3️⃣ Create System Configuration
```
sudo bash -c 'cat > /etc/sysctl.d/99-popcache.conf << EOL
net.ipv4.ip_local_port_range = 1024 65535
net.core.somaxconn = 65535
net.ipv4.tcp_low_latency = 1
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.core.wmem_max = 16777216
net.core.rmem_max = 16777216
EOL'
sudo sysctl -p /etc/sysctl.d/99-popcache.conf
```
```
sudo bash -c 'cat > /etc/security/limits.d/popcache.conf << EOL
*    hard nofile 65535
*    soft nofile 65535
EOL'
```
Now close ur VPS or WSL or Ubuntu Then Open again ur VPS or WSL

4️⃣ Make Directory (create folder to be used for download cache)
```
sudo mkdir -p /opt/popcache
sudo mkdir -p /opt/popcache/logs
```
```
cd /opt/popcache
```

For VPS Only
```
apt install screen -y
```
```
screen -S pipetestnet
```
- PRESS CTRL+A+D (to run ur node continuously)
- To check ur Node Again
```
screen -r pipetestnet
```

5️⃣ Download Pipe Binaries
1. Visit: [Download](https://download.pipe.network/) file (use invite code from email)
2. Once Downloaded use termius SFTP feature to drag and drop the downloaded file in `/opt/popcache` you created in step 4

For Local PC
```
sudo cp -r /mnt/c/Users/ASUS/Downloads/File /opt/popcache/
```
Replace "c/Users/ASUS/Downloads/File" to your Actual File Path in ur System

For VPS (Run in Powershell or Command Prompt)
```
scp -r "C:\Users\ASUS\Downloads\nohos" root@VPS_IP:/opt/popcache/
```
Replace "C:\Users\ASUS\Downloads\File" to your Actual File Path in ur System and VPS_IP with ur Actual VPS IP
```
sudo tar -xzf pop-v0.3.0-linux-*.tar.gz
```
```
sudo chmod +x /opt/popcache/pop
```

6️⃣ Setup Config File
```
nano config.json
```
```
{
  "pop_name": "your-pop-name",
  "pop_location": "Your Location, Country",
  "invite_code": "Enter your Invite Code",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 0
  },
  "cache_config": {
    "memory_cache_size_mb": 4096,
    "disk_cache_path": "./cache",
    "disk_cache_size_gb": 100,
    "default_ttl_seconds": 86400,
    "respect_origin_headers": true,
    "max_cacheable_size_mb": 1024
  },
  "api_endpoints": {
    "base_url": "https://dataplane.pipenetwork.com"
  },
  "identity_config": {
    "node_name": "your-node-name",
    "name": "Your Name",
    "email": "your.email@example.com",
    "website": "https://your-website.com",
    "discord": "your_discord_username",
    "telegram": "your_telegram_handle",
    "solana_pubkey": "YOUR_SOLANA_WALLET_ADDRESS_FOR_REWARDS"
  }
}
```

- `pop-location`: location of VPS --> Command to Check --> `realpath --relative-to /usr/share/zoneinfo /etc/localtime`
- `website`: Anything you prefer (can use github profile)



7️⃣ Creating a Systemd Service File
```
sudo bash -c 'cat > /etc/systemd/system/popcache.service << EOL
[Unit]
Description=POP Cache Node
After=network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/opt/popcache
ExecStart=/opt/popcache/pop
Restart=always
RestartSec=5
LimitNOFILE=65535
StandardOutput=append:/opt/popcache/logs/stdout.log
StandardError=append:/opt/popcache/logs/stderr.log
Environment=POP_CONFIG_PATH=/opt/popcache/config.json

[Install]
WantedBy=multi-user.target
EOL'
```

8️⃣ Run Node
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable popcache
```
```
sudo systemctl start popcache
```

# Open Another Window for WSL or VPS

## Monitor your Node Status & Logs
```
sudo systemctl status popcache
```
```
sudo journalctl -u popcache
```

## Check Points & Status from Dashboard - https://dashboard.pipenetwork.com/node-lookup


## 🔶For Next Day Run This Command

#1 Open WSL and Put this Command 
```
cd /opt/popcache
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable popcache
```
```
sudo systemctl start popcache
```

## Need to Free Your 8003 Port

### Identify the Process Using Port 8003
```
sudo ss -tulpn | grep 8003
```

Example - ``` LISTEN  0  128  0.0.0.0:0380  0.0.0.0:*  users:(("nginx",pid=1234,fd=6)) ```

### Terminate the Process by PID
```
sudo kill -9 1234
```

### Kill All Processes Using Port 8003
```
sudo fuser -k 8003/tcp
```

Reference Video How to Free 8003 or any port - https://youtu.be/4iP4GvLfCrU?t=229

## Delete Pipe node
```
cd /data
rm -f pop  # Deletes the 'pop' binary  
rm -rf download_cache  # Deletes the 'download_cache' folder  
rm -f node_info.json  # Deletes the 'node_info.json' file
```
