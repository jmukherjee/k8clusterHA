# -*- mode: ruby -*-
# vi: set ft=ruby :

N_MASTER = 1
N_WORKER = 1
ID = "#{rand(01..99)}"
TS = Time.now.to_i
IP_BASE = "192.168.10"


scriptHAProxy = <<SCRIPT
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
SCRIPT

scriptRSACallee = <<SCRIPT
cat /vagrant/rsa/rsa_pub >> /root/.ssh/authorized_keys
sudo -H -u vagrant bash -c 'cat /vagrant/rsa/rsa_pub >> /home/vagrant/.ssh/authorized_keys'
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = 'ubuntu/bionic64'
  config.vm.synced_folder ".", "/vagrant", disabled: false

  config.vm.define "vlb", primary: true do |vlb|
    vlb.vm.hostname = "vlb"
    vlb.vm.network :private_network, ip: "192.168.10.10"
    vlb.vm.network "forwarded_port", guest: 80, host: 3000
    vlb.vm.network "forwarded_port", guest: 8404, host: 8404
    vlb.vm.provider "virtualbox" do |vb|
      vb.name = "k8HA-VLB"

      vb.memory = "1024"
      vb.cpus = 1

      vb.customize ["modifyvm", :id, "--ioapic", "on"]
      vb.gui = false
    end
    vlb.vm.provision "shell", inline: <<-SHELL
      apt-get update
      echo "------- INSTALL: HA Proxy -------"
      apt-get install -y haproxy
    SHELL
    vlb.vm.provision "shell", :inline => scriptHAProxy
    vlb.vm.provision "shell", :inline => scriptRSACaller
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
      k8m.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y apache2
        echo "<H1>K8Master#{i+1} [#{MSTR_IP}]</H2><h2>(#{ID}) #{TS}</h2>" > /var/www/html/index.html
      SHELL
      k8m.vm.provision "shell", :inline => scriptRSACallee
      k8m.trigger.after :up do |trigger|
        trigger.run = { inline:
          "vagrant ssh vlb -- ssh-keyscan -t rsa #{MSTR_IP} >> ~/.ssh/known_hosts"
        }
      end
    end
  end

  N_WORKER.times do |i|
    WRKR_IP = "#{IP_BASE}.#{20+i+1}"
    WRKR_NAME = "kworker#{i+1}"
    config.vm.define "#{WRKR_NAME}" do |k8w|
      k8w.vm.hostname = "#{WRKR_NAME}"
      k8w.vm.network :private_network, ip: "#{WRKR_IP}"
      k8w.vm.provider "virtualbox" do |vb|
        vb.name = "k8HA-#{WRKR_NAME}"

        vb.memory = "1024"
        vb.cpus = 1

        vb.customize ["modifyvm", :id, "--ioapic", "on"]
        vb.gui = false
      end
      k8w.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y apache2
        echo "<H1>k8worker#{i+1} [#{WRKR_IP}]</H2><h2>(#{ID}) #{TS}</h2>" > /var/www/html/index.html
      SHELL
      # k8w.vm.provision "shell", :inline => scriptRSACallee
      # k8w.trigger.after :up do |trigger|
      #   trigger.run = { inline:
      #     "vagrant ssh vlb -- ssh-keyscan -t rsa #{WRKR_IP} >> ~/.ssh/known_hosts"
      #   }
      # end
    end
  end
end