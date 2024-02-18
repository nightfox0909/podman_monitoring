# podman_monitoring

## Enable podman socket
```bash
sudo systemctl enable --now podman.socket
sudo chmod 666 /run/podman/podman.sock
```

```bash
podman system service --time=0 unix:///run/user/1000/podman/podman.sock &
```

## Run stack
```bash
sudo podman-compose -f compose.yaml up -d
```

## grafana ui
<IP>:3000
