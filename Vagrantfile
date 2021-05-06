# -*- mode: ruby -*-
# vi: set ft=ruby :

N_MASTER = 1
N_WORKER = 2
ID = "#{rand(01..99)}"
TS = Time.now.to_i
IP_BASE = "192.168.10"

scriptApache = <<SCRIPT
apt-get update
apt-get install -y apache2
echo "<H1>$1</H2><h2>(#{ID}) #{TS}</h2>" > /var/www/html/index.html
SCRIPT

scriptHAProxy = <<SCRIPT
apt-get update
echo "------- INSTALL: HA Proxy -------"
apt-get install -y haproxy

echo "------- CONFIGURE: HA Proxy -------"
cat >> /etc/haproxy/haproxy.cfg <<EOF
frontend main
  bind *:80
  use_backend k8masters
backend k8masters
$(for var in {1..#{N_MASTER}}; do
echo "server k8m$var 192.168.10.$(($var+10)):80 check"
done)
listen stats
  bind *:8404
  stats enable
  stats uri /monitor
EOF

echo "------- RESTART: HA Proxy -------"
systemctl restart haproxy

haproxy -vv
SCRIPT

scriptRSACaller = <<SCRIPT
echo "------- ROOT: Setup RSA -------"
cp /vagrant/rsa/rsa_pvt /root/.ssh/id_rsa
cp /vagrant/rsa/rsa_pub /root/.ssh/id_rsa.pub
chmod 600 /root/.ssh/id_rsa

echo "------- USER: Setup RSA -------"
sudo -H -u vagrant bash -c 'cp /vagrant/rsa/rsa_pvt /home/vagrant/.ssh/id_rsa'
sudo -H -u vagrant bash -c 'cp /vagrant/rsa/rsa_pub /home/vagrant/.ssh/id_rsa.pub'
chmod 600 /home/vagrant/.ssh/id_rsa
chown vagrant:vagrant /home/vagrant/.ssh/id_rsa*

# sudo -H -u vagrant bash -c 'ssh-agent bash'
# sudo -H -u vagrant bash -c 'ssh-add /home/vagrant/.ssh/id_rsa'
echo "SSH Connect: ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no vagrant@<name/ip>"
SCRIPT

scriptRSACallee = <<SCRIPT
cat /vagrant/rsa/rsa_pub >> /root/.ssh/authorized_keys
sudo -H -u vagrant bash -c 'cat /vagrant/rsa/rsa_pub >> /home/vagrant/.ssh/authorized_keys'
SCRIPT

scriptHost = <<SCRIPT
cat >> /etc/hosts <<EOF
192.168.10.10  vlb
$(for var in {1..#{N_MASTER}}; do
echo "192.168.10.$(($var+10))  kmaster$var"
done)
$(for var in {1..#{N_WORKER}}; do
echo "192.168.10.$(($var+20))  kslave$var"
done)
EOF
SCRIPT

scriptDashboardNodePort = <<SCRIPT
cat > k8_db_nodeport.yaml <<EOF
apiVersion: v1
kind: Service
metadata:
  namespace: kubernetes-dashboard
  name: k8-svc-dashboard-nodeport
  labels:
    k8s-app: kubernetes-dashboard
spec:
  type: NodePort
  ports:
  - port: 8443
    nodePort: 30002
    targetPort: 8443
    protocol: TCP
  selector:
    k8s-app: kubernetes-dashboard
EOF
SCRIPT

scriptK8Master = <<SCRIPT
ADDR_EXT=$(ifconfig enp0s8 | grep 'inet ' | xargs | cut -d " " -f 2)

echo "------------- Configuring Kubernetes -------------"
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=$ADDR_EXT --apiserver-cert-extra-sans=$ADDR_EXT --node-name kmaster
mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown vagrant:vagrant /home/vagrant/.kube/config

export KUBECONFIG=/etc/kubernetes/admin.conf

echo "------------- Deploying Calico -------------"
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubeadm token create --print-join-command > /vagrant/join-#{TS}.sh

sleep 5

echo "------------- Installing Dashboard + Nodeport -------------"
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
kubectl apply -f k8_db_nodeport.yaml
kubectl get pods -A
kubectl create serviceaccount dashboard -n default
kubectl create clusterrolebinding dashboard-admin -n default --clusterrole=cluster-admin --serviceaccount=default:dashboard
kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous
#kubectl label nodes kslave01 kubernetes.io/role=worker
kubectl proxy &
sleep 5

sed -i -e 's/- --port=0/#- --port=0/' /etc/kubernetes/manifests/kube-scheduler.yaml
sed -i -e 's/- --port=0/#- --port=0/' /etc/kubernetes/manifests/kube-controller-manager.yaml
systemctl restart kubelet

echo "------------- Validating Cluster -------------"
echo "External Link: https://$ADDR_EXT:30002/"
tail -10 /var/log/syslog
kubectl cluster-info
kubectl get pods -o wide --all-namespaces
kubectl get nodes -o wide
kubectl get cs

echo "------------- Dashboard Details -------------"
echo "External Link: https://$ADDR_EXT:30002/#/login"
kubectl -n kubernetes-dashboard describe service kubernetes-dashboard
kubectl describe secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}")
SCRIPT

scriptK8Slave = <<SCRIPT
apt-get install -y sshpass

echo "------------- Joining Cluster -------------"
chmod +x /vagrant/join-#{TS}.sh
bash /vagrant/join-#{TS}.sh
#kubectl label nodes kslave01 kubernetes.io/role=worker
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = 'ubuntu/bionic64'
  config.vm.synced_folder ".", "/vagrant", disabled: false

  config.trigger.after :up do |trigger|
    trigger.name = "POST ALL Provision"
    trigger.info = "Cluster is up!! SSH Connect: ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no vagrant@<name/ip>"
    #trigger.run_remote = {inline: "pg_dump dbname > /vagrant/outfile"}
  end

  VLB_IP = "#{IP_BASE}.#{10}"
  VLB_NAME = "vlb"
  config.vm.define "#{VLB_NAME}", primary: true do |vlb|
    vlb.vm.hostname = "#{VLB_NAME}"
    vlb.vm.network :private_network, ip: "#{VLB_IP}"
    vlb.vm.network "forwarded_port", guest: 80, host: 3000
    vlb.vm.network "forwarded_port", guest: 8404, host: 8404
    vlb.vm.provider "virtualbox" do |vb|
      vb.name = "k8HA-VLB"

      vb.memory = "1024"
      vb.cpus = 1

      vb.customize ["modifyvm", :id, "--ioapic", "on"]
      vb.gui = false
    end
    
    vlb.vm.provision "VLB: Hostname ---------->", type: "shell", inline: scriptHost
    vlb.vm.provision "VLB: HAProxy setup ----->", type: "shell", inline: scriptHAProxy
    vlb.vm.provision "RSA Setup: VLB Caller -->", type: "shell", inline: scriptRSACaller
  end

  N_MASTER.times do |i|
    MSTR_IP = "#{IP_BASE}.#{10+i+1}"
    MSTR_NAME = "kmaster#{i+1}"
    config.vm.define "#{MSTR_NAME}" do |k8m|
      k8m.vm.hostname = "#{MSTR_NAME}"
      k8m.vm.network :private_network, ip: "#{MSTR_IP}"
      k8m.vm.provider "virtualbox" do |vb|
        vb.name = "k8HA-#{MSTR_NAME}"

        vb.memory = "2048"
        vb.cpus = 2

        vb.customize ["modifyvm", :id, "--ioapic", "on"]
        vb.gui = false
      end
      k8m.vm.provision "Master Setup: Apache -------->", type: "shell", inline: scriptApache, args: "K8Master#{i+1}-[#{MSTR_IP}]"
      k8m.vm.provision "Master Setup: Bootstrap ----->", type: "shell",   path: "bootstrap-k8.sh"
      k8m.vm.provision "Master: Hostname ------------>", type: "shell", inline: scriptHost
      k8m.vm.provision "RSA Setup: Master Caller ---->", type: "shell", inline: scriptRSACaller
      k8m.vm.provision "RSA Setup: Master Callee ---->", type: "shell", inline: scriptRSACallee
      k8m.vm.provision "Master: Dashboard Nodeport -->", type: "shell", inline: scriptDashboardNodePort
      k8m.vm.provision "Master: K8 config ----------->", type: "shell", inline: scriptK8Master
    end
  end

  N_WORKER.times do |i|
    config.vm.define "kworker#{i+1}" do |k8w|
      WRKR_IP = "#{IP_BASE}.#{20+i+1}"
      WRKR_NAME = "kworker#{i+1}"
      k8w.vm.hostname = "kworker#{i+1}"
      k8w.vm.network :private_network, ip: "#{WRKR_IP}"
      k8w.vm.provider "virtualbox" do |vb|
        vb.name = "k8HA-kworker#{i+1}"

        vb.memory = "1024"
        vb.cpus = 1

        vb.customize ["modifyvm", :id, "--ioapic", "on"]
        vb.gui = false
      end

      k8w.vm.provision "Worker Setup: Apache -------->", type: "shell", inline: scriptApache, args: "K8Worker#{i+1}-[#{WRKR_IP}]"
      k8w.vm.provision "Worker Setup: Bootstrap ----->", type: "shell",   path: "bootstrap-k8.sh"
      k8w.vm.provision "Worker: Hostname ------------>", type: "shell", inline: scriptHost
      k8w.vm.provision "RSA Setup: Worker Callee ---->", type: "shell", inline: scriptRSACallee
      k8w.vm.provision "Worker: K8 config ----------->", type: "shell", inline: scriptK8Slave
    end
  end
end