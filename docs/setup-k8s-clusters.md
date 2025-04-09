# Hướng Dẫn Cài Đặt Cụm Kubernetes

## Cấu hình tài nguyên

| Hostname     | OS           | IP            | RAM (tối thiểu) | CPU (tối thiểu) |
| ------------ | ------------ | ------------- | --------------- | --------------- |
| k8s-master-1 | Ubuntu 22.04 | 192.168.0.211 | 4G              | 2               |
| k8s-master-2 | Ubuntu 22.04 | 192.168.0.212 | 4G              | 2               |
| k8s-master-3 | Ubuntu 22.04 | 192.168.0.213 | 4G              | 2               |

> **Note**: Chú ý xem kỹ bài giảng để cài đặt cụm phù hợp.

## Thực hiện trên cả 3 servers

### Thêm hosts

```bash
$ vi /etc/hosts/
```

Nội dung cấu hình:

```
192.168.0.211 k8s-master-1
192.168.0.212 k8s-master-2
192.168.0.213 k8s-master-3
```

### Cập nhật và nâng cấp hệ thống

```bash
$ sudo apt update -y && sudo apt upgrade -y
```

### Tạo user devops và chuyển sang user devops

```bash
$ adduser devops
$ su devops
$ cd /home/devops
```

### Tắt swap

```bash
$ sudo swapoff -a
$ sudo sed -i '/swap.img/s/^/#/' /etc/fstab
```

### Cấu hình module kernel

```bash
$ vi /etc/modules-load.d/containerd.conf
```

Nội dung sau:

```
overlay
br_netfilter
```

### Tải module kernel

```bash
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```

### Cấu hình hệ thống mạng

```bash
$ echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
$ echo "net.bridge.bridge-nf-call-iptables = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
$ echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
```

### Áp dụng cấu hình sysctl

```bash
$ sudo sysctl --system
```

### Cài đặt các gói cần thiết và thêm kho Docker

```bash
$ sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

### Cài đặt containerd

```bash
$ sudo apt update -y
$ sudo apt install -y containerd.io
```

### Cấu hình containerd

```bash
$ containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
$ sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```

### Khởi động containerd

```bash
$ sudo systemctl restart containerd
$ sudo systemctl enable containerd
```

### Thêm kho lưu trữ Kubernetes

```bash
$ echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### Cài đặt các gói Kubernetes

```bash
$ sudo apt update -y
$ sudo apt install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

> Xem tiếp bài giảng để cài đặt K8s cluster phù hợp (xác định mô hình cụm cần cài đặt và reset cụm nếu cài không như mong muốn)

### Khối lệnh reset cụm khi đã khởi tạo cụm

```bash
$ sudo kubeadm reset -f
$ sudo rm -rf /var/lib/etcd
$ sudo rm -rf /etc/kubernetes/manifests/*
```

## Mô hình đầu tiên: 1 master 2 worker

### Thực hiện trên server k8s-master-1

```bash
$ sudo kubeadm init
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.0/manifests/calico.yaml
$ kubectl token create --print-join-command
```

### Thực hiện trên server k8s-master-2 và k8s master-3

```bash
$ sudo kubeadm join 192.168.1.111:6443 --token your_token --discovery-token-ca-cert-hash your_sha
```

## Mô hình thứ hai: 3 master (worker) – xem video bài giảng để hiểu

### Thực hiện trên server k8s-master-1

```bash
$ sudo kubeadm init --control-plane-endpoint "192.168.1.111:6443" --upload-certs
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.0/manifests/calico.yaml
```

### Thực hiện trên server k8s-master-2 và k8s-master-3

```bash
$ sudo kubeadm join 192.168.1.111:6443 --token your_token --discovery-token-ca-cert-hash your_sha --control-plane --certificate-key your_cert
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
