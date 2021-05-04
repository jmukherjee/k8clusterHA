# -*- mode: ruby -*-
# vi: set ft=ruby :


# Vagrant.configure("2") do |config|
#   config.vm.box = "base"
#   # config.vm.box_check_update = false
#   # config.vm.network "forwarded_port", guest: 80, host: 8080
#   # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
#   # config.vm.network "private_network", ip: "192.168.33.10"
#   # config.vm.network "public_network"
#   # config.vm.provider "virtualbox" do |vb|
#   #   # Display the VirtualBox GUI when booting the machine
#   #   vb.gui = true
  
#   #   # Customize the amount of memory on the VM:
#   #   vb.memory = "1024"
#   # end
#   # config.vm.provision "shell", inline: <<-SHELL
#   #   apt-get update
#   #   apt-get install -y apache2
#   # SHELL
# end

N_MASTER = 1

scriptHAProxy = <<SCRIPT
cat >> /etc/haproxy/haproxy.cfg <<EOF
frontend main
  bind *:80
  use_backend apps

backend apps
$(for var in {1..#{N_MASTER}}; do
echo "server app$var 192.168.10.1$var:80 check"
done)

listen stats
  bind *:8404
  stats enable
  stats uri /monitor
EOF

systemctl restart haproxy
SCRIPT

Vagrant.configure("2") do |config|

  config.vm.define "loadbalancer", primary: true do |vlb|
    vlb.vm.box = 'ubuntu/bionic64'
    vlb.vm.hostname = "loadbalancer"
    vlb.vm.network :private_network, ip: "192.168.10.10"
    vlb.vm.network "forwarded_port", guest: 80, host: 3000
    vlb.vm.network "forwarded_port", guest: 8404, host: 8404
    vlb.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y haproxy
    SHELL
    vlb.vm.provision "shell", :inline => scriptHAProxy
  end

  N_MASTER.times do |i|
    config.vm.define "app-#{i+1}" do |app|
      app.vm.box = 'ubuntu/bionic64'
      app.vm.hostname = "app-#{i+1}"
      app.vm.network :private_network, ip: "192.168.10.#{10+i+1}"
      app.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y apache2
        echo "<H1>App-#{i+1}</H2>" > /var/www/html/index.html
      SHELL
      app.trigger.after :up do |trigger|
        trigger.run = { inline: 
          "vagrant ssh loadbalancer -- cp .vagrant/machines/app-#{i+1}/virtualbox/private_key ~/.ssh/id_rsa"
          #{}"vagrant ssh loadbalancer -- echo 'hello' > ~/hello.txt"
        }
      end
    end
  end
end