# -*- mode: ruby -*-
# vi: set ft=ruby :

# This script to install Kubernetes will get executed after we have provisioned the box 
$script = <<-SCRIPT

# Install kubernetes
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl jq

# kubelet requires swap off
swapoff -a
# keep swap off after reboot
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Hackily ensure kubelet uses the same CGroup driver as Docker
# sed -i '/ExecStart=/a Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
sed -i '0,/ExecStart=/s//Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"\n&/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# Set up Kubernetes
NODENAME=$(hostname -s)
kubeadm init --node-name $NODENAME --pod-network-cidr=10.244.0.0/16

# Allow pods to run on the master node (since we only have 1)
kubectl --kubeconfig /etc/kubernetes/admin.conf taint nodes --all node-role.kubernetes.io/master-

# Set /proc/sys/net/bridge/bridge-nf-call-iptables to 1 by running sysctl net.bridge.bridge-nf-call-iptables=1 to pass bridged IPv4 traffic to iptables’ chains.
sudo sysctl net.bridge.bridge-nf-call-iptables=1

# Install Flannel for CNI
kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

# Install the prometheus-operator
kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.24.0/bundle.yaml

# Wait for the operator to install the CRDs it uses for managing Prometheus instances
echo Waiting for operator to install CRDs
until kubectl --kubeconfig /etc/kubernetes/admin.conf api-versions | grep -q monitoring.coreos.com/v1; do sleep 1; printf "."; done; echo "ready!"

# Install the latest version of Prometheus using the prometheus-operator
kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f /vagrant/manifests/prometheus.yaml

# Install the latest version of Grafana with a preconfigured dashboard
kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f /vagrant/manifests/grafana.yaml

# Wait for the Grafana dashboard to become available
echo Waiting for Grafana dashboard to become available
until kubectl --kubeconfig /etc/kubernetes/admin.conf get deployment grafana -o json | jq -e '.status.readyReplicas >= 1' > /dev/null 2>&1; do sleep 1; printf "."; done; echo "ready!"

# Set up admin creds for the vagrant user
echo Copying credentials to /home/vagrant...
sudo --user=vagrant mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config
SCRIPT

Vagrant.configure("2") do |config|
  # Specify your hostname if you like
  # config.vm.hostname = "name"
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.network "private_network", type: "dhcp"
  config.vm.network "forwarded_port", guest: 30030, host: 3000
  config.vm.network "forwarded_port", guest: 30090, host: 9090
  config.vm.provision "docker"
  config.vm.provision "shell", inline: $script
end