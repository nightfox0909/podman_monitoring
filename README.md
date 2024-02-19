# podman_monitoring
This is a podman & node monitoring stack fully based off on conda and does not require sudo permissions. It uses two prometheus exporters
1. https://github.com/containers/prometheus-podman-exporter
2. https://github.com/prometheus/node_exporter

Uses the following awesome dashboards
1. https://grafana.com/grafana/dashboards/1860-node-exporter-full
2. https://github.com/containers/prometheus-podman-exporter

This repo and stack is designed to work out of the box and provide an quick initial dashboard. It automatically sets up the exporters, configures prometheus and points grafana at it. 

## Step 1: Install podman
configure conda channels

```bash
conda config --add channels conda-forge
conda config --set channel_priority strict
```

install podman
```bash
conda install podman
```
restart your current terminal. 

## Step 2: Configure podman
### 2.1 Add helper binaries folder to find rootlessport
When it starts first, podman will have issues finding rootlessport since its not on the PATH. To fix this, 
Edit  /home/sp/miniconda3/share/containers/containers.conf

Add the sections below
```bash
[engine]
helper_binaries_dir = [
  "/home/sp/miniconda3/libexec/podman"
]
```

### 2.2 Change network backend to netavark
First, check what is the current network backend.
```bash
podman info | grep networkBackend
```
**If you see cni, continue with the steps below. If you see netavark, skip this step as you already have the correct backend and pick up from enable podman socket section**

The default CNI network stack does not have dns resolution. To enable this, we need to install netavark. 

#### 2.2.1 Option 1 : Build netavark and aadvark-dns
This method requires sudo permissions to install build dependencies. You could build for a target machine and then deploy the binaries without sudo. Alternatively you can use Option 2 if you have glibc version 2.32 and above. 

Install pre-requisites
1. Rustc
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
```
2. Deps
```bash
sudo apt-get install build-essential protobuf-compiler
```
3. Clone and build netavark
```bash
git clone https://github.com/containers/netavark.git
cd netavark
make
mv bin/netavark /home/sp/miniconda3/libexec/podman/
```
5. Clone and build aadvark-dns
```bash
git clone https://github.com/containers/aardvark-dns.git
cd aardvark-dns
make
mv bin/aardvark-dns /home/sp/miniconda3/libexec/podman/
```

#### 2.2.2 Option 2: Download builds directly
```bash
wget https://github.com/containers/netavark/releases/download/v1.10.3/netavark.gz
gzip -d netavark.gz
chmod +x netavark
mv netavark /home/sp/miniconda3/libexec/podman/
wget wget https://github.com/containers/aardvark-dns/releases/download/v1.10.0/aardvark-dns.gz
gzip -d aardvark-dns.gz
chmod +x aardvark-dns
mv aardvark-dns /home/sp/miniconda3/libexec/podman/
```

### 2.3 Change default engine from cni to netavark
edit network_engine in /home/sp/miniconda3/share/containers/containers.conf
```bash
[network]
network_backend = "netavark"
```

### 2.4 Reset podman
This step is destructive and removes all existing images, containers and networks. Proceed only after backups are created.
```bash
podman system reset --force
```

## Step 3: Enable podman socket
A socket is how the podman monitoring component communicates with the containers and images. Since podman is daemonless, we need to create this socket and give read permissions on it.

To do this, we first setup some systemd services. copy the systemd files from the folder systemd to /usr/lib/systemd/user/
```bash
cp systemd/. /usr/lib/systemd/user/
```
enable the new files and enable the podman listener socket.
```bash
systemctl --user daemon-reload
systemctl --user start podman.socket
```

## Step 4: Run stack
```bash
podman compose -f compose.yaml up -d
```

## Step 5: View grafana ui
You are all done and can view the dashboard at the link below. 
IP_ADDRESS_OF_MAACHINE:3000
