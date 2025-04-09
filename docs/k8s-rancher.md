# Quản lý các cụm K8s

### Setup rancher (Kubernetes)

- Format disk
  ```
  $ sudo mkfs.ext4 -m 0 /dev/sdb
  ```
- Gán disk (ổ cứng) vào trong thư mục /data

  ```bash
  $ mkdir /data
  $ echo "/dev/sdb /data ext4 defaults 0 0" | sudo tee -a /etc/fstab
  $ mount -a
  $ sudo df -h
  ```

- `docker-compose.yml` tạo Rancher

  ```
  version: '3'
  services:
  rancher-server:
    image: rancher/rancher:v2.9.2
    container_name: rancher-server
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
  volumes: - /data/rancher/data:/var/lib/rancher
  privileged: true
  ```

- Lấy mật khẩu Rancher

  ```
  $ docker logs rancher-server 2>&1 | grep "Bootstrap Password:"
  ```
