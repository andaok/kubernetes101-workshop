# -*- mode: ruby -*-
# vi: set ft=ruby :

hosts = {
	"master01" => "192.168.33.101",
	"node01" => "192.168.33.110"
}

$script = <<SCRIPT
swapoff -a
echo "root:root" | chpasswd

cat > /etc/apt/sources.list.d/kubernetes.list << EOF
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg \
       | apt-key add -
apt-get update -y
apt-get upgrade -y
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
apt-get install docker-ce=18.03.0~ce-0~ubuntu -y --allow-downgrades
apt-get install -y kubeadm kubelet kubectl
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sed -i '0,/ExecStart=/s//Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"\n&/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables

cat > /etc/hosts << EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
192.168.33.101 master01
192.168.33.110 node01
EOF

docker version
kubeadm version
kubelet --version

SCRIPT


Vagrant.configure("2") do |config|
  hosts.each do |name, ip|
    config.vm.define name do |machine|
      machine.vm.box = "bento/ubuntu-16.04"
      machine.vm.hostname = name
      machine.vm.network :private_network, ip: ip
      machine.vm.provision "shell", inline: $script
      machine.vm.provider "virtualbox" do |v|
          v.name = name
          v.customize ["modifyvm", :id, "--memory", 4048]
      end
    end
  end
end