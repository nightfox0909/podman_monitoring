# podman_monitoring

## Enable podman socket
sudo systemctl enable --now podman.socket
sudo chmod 666 /run/podman/podman.sock

## Run stack
sudo podman-compose -f compose.yaml up -d

## grafana ui
<IP>:3000
