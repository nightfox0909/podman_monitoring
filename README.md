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

### change network backend to netavark
The default CNI network stack does not have dns resolution. To enable this, we need to install netavark. 

edit network_engine in /home/sp/miniconda3/share/containers/containers.conf
```bash
[network]
network_backend = "netavark"
```

download netavark and aadvark-dns to /home/sp/miniconda3/libexec/podman
```bash
wget https://github.com/containers/netavark/releases/download/v1.10.3/netavark.gz
gzip -d netavark.gz
chmod +x netavark
mv netavark /home/sp/miniconda3/libexec/podman/
```

```bash
wget wget https://github.com/containers/aardvark-dns/releases/download/v1.10.0/aardvark-dns.gz
gzip -d aardvark-dns.gz
chmod +x aardvark-dns
mv aardvark-dns /home/sp/miniconda3/libexec/podman/
```

reset podman. This step is destructive and removes all existing images, containers and networks. Proceed only after backups are created.
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
sudo podman-compose -f compose.yaml up -d
```

## grafana ui
<IP>:3000
