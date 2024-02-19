# podman_monitoring
This is a podman stack fully based off on conda and does not require sudo permissions

current issues : podman socket needs to be made into a systemd process. right now, the socket process must be kept open while the stack runs. 

## install podman
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

## configure podman
### Add helper binaries folder to find rootlessport
When it starts first, podman will have issues finding rootlessport since its not on the PATH. To fix this, 
Edit  /home/sp/miniconda3/share/containers/containers.conf

Add the sections below
```bash
[engine]
helper_binaries_dir = [
  "/home/sp/miniconda3/libexec/podman"
]
```

### Change network backend to netavark
The default CNI network stack does not have dns resolution. To enable this, we need to install netavark. 

#### Option 1 : Build netavark and aadvark-dns
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

#### Option 2: Download builds directly
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

edit network_engine in /home/sp/miniconda3/share/containers/containers.conf
```bash
[network]
network_backend = "netavark"
```

### Reset podman
This step is destructive and removes all existing images, containers and networks. Proceed only after backups are created.
```bash
podman system reset --force
```

## Enable podman socket
A socket is how the podman monitoring component communicates with the containers and images. Since podman is daemonless, we need to create this socket and give read permissions on it.
```bash
podman system service --time=0 unix:///run/user/1000/podman/podman.sock
chmod 666 /run/user/1000/podman/podman.sock
```

## Run stack
```bash
podman compose -f compose.yaml up -d
```

## grafana ui
<IP>:3000
