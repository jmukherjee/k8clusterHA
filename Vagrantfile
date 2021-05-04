# -*- mode: ruby -*-
# vi: set ft=ruby :

N_MASTER = 1

scriptHAProxy = <<SCRIPT
cat >> /etc/haproxy/haproxy.cfg <<EOF
frontend main
  bind *:80
  use_backend apps

backend apps
$(for var in {1..#{N_MASTER}}; do
echo "server app$var 192.168.10.$(($var+10)):80 check"
done)

listen stats
  bind *:8404
  stats enable
  stats uri /monitor
EOF

systemctl restart haproxy
SCRIPT

scriptRSACaller = <<SCRIPT
cp /vagrant/rsa/rsa_pvt /root/.ssh/id_rsa
cp /vagrant/rsa/rsa_pub /root/.ssh/id_rsa.pub
chmod 600 /root/.ssh/id_rsa

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

  config.vm.define "loadbalancer", primary: true do |vlb|
    vlb.vm.hostname = "loadbalancer"
    vlb.vm.network :private_network, ip: "192.168.10.10"
    vlb.vm.network "forwarded_port", guest: 80, host: 3000
    vlb.vm.network "forwarded_port", guest: 8404, host: 8404
    vlb.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y haproxy
    SHELL
    vlb.vm.provision "shell", type: "shell", :inline => scriptHAProxy
    vlb.vm.provision "shell", type: "shell", :inline => scriptRSACaller
  end

  N_MASTER.times do |i|
    config.vm.define "app-#{i+1}" do |app|
      app.vm.hostname = "app-#{i+1}"
      app.vm.network :private_network, ip: "192.168.10.#{10+i+1}"
      app.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y apache2
        echo "<H1>App-#{i+1}</H2>" > /var/www/html/index.html
      SHELL
      app.vm.provision "shell", :inline => scriptRSACallee
      app.trigger.after :up do |trigger|
        trigger.run = { inline: 
          #"vagrant ssh loadbalancer -- cp .vagrant/machines/app-#{i+1}/virtualbox/private_key ~/.ssh/id_rsa"
          "vagrant ssh loadbalancer -- ssh-keyscan -t rsa 192.168.10.#{10+i+1} >> ~/.ssh/known_hosts"
        }
      end
    end
  end
end