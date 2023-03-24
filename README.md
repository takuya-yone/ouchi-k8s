# ouchi-k8s

sudo hostnamectl set-hostname k8s-m1.local
sudo hostnamectl set-hostname k8s-n1.local
sudo hostnamectl set-hostname k8s-n2.local

sudo nano /etc/hosts

192.168.0.101      k8s-m1.local
192.168.0.102      k8s-n1.local
192.168.0.103      k8s-n2.local


sudo nano /etc/dhcpcd.conf

interface wlan0
static ip_address=192.168.0.101/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1


interface wlan0
static ip_address=192.168.0.102/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1

interface wlan0
static ip_address=192.168.0.103/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1

sudo nano /etc/netplan/99-network.yaml

network:
  version: 2
  renderer: networkd
  wifis:
    wlan0:
      dhcp4: false
      dhcp6: false
      addresses:
        - 192.168.0.101/24
      gateway4: 192.168.0.1
      nameservers:
        addresses:
          - 192.168.0.1

network:
  version: 2
  renderer: networkd
  wifis:
    wlan0:
      dhcp4: false
      dhcp6: false
      addresses:
        - 192.168.0.102/24
      gateway4: 192.168.0.1
      nameservers:
        addresses:
          - 192.168.0.1


network:
  version: 2
  renderer: networkd
  wifis:
    wlan0:
      dhcp4: false
      dhcp6: false
      addresses:
        - 192.168.0.103/24
      gateway4: 192.168.0.1
      nameservers:
        addresses:
          - 192.168.0.1

ssh-copy-id 192.168.0.101
ssh-copy-id 192.168.0.102
ssh-copy-id 192.168.0.103


sudo apt update
sudo apt upgrade
sudo apt-get install build-essential procps curl file git vim linux-modules-extra-raspi




sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg


curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

sudo rm /etc/containerd/config.toml
sudo systemctl restart containerd


sudo kubeadm init --control-plane-endpoint=k8s-m1.local:6443 --pod-network-cidr=10.244.0.0/16 --upload-certs

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml


kubeadm join k8s-m1.local:6443 --token 4ezrqy.t4yvvy4zhmu1r1aw \
	--discovery-token-ca-cert-hash sha256:9c514f1e365bd1707ee64b3e25fe263aa432bbdae1d4570cf13e44ca61838af6
