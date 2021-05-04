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
echo "server app$var 192.168.10.$(($var+10)):80 check"
done)

listen stats
  bind *:8404
  stats enable
  stats uri /monitor
EOF

systemctl restart haproxy
SCRIPT

scriptPVTKey = <<SCRIPT
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAtwSJSlbq4V3lSuNFCczr8GVOYLRxpZegYdGI47mrknpSA5/V
qJ96JnWvi75p6X/MbZcnZSd0TuJdLmgOAKKCHsVXpuqhGsGJHyF6nOH3Mp9qRKgN
RH3F2unIlkr9lNYGNoh7LJnBpOY9jexjJikMeGxjHRmwjw2bH68HsNzeaR7xhBlX
5+5tiys4nHdDcTt7d9ZjOPpNcOl2vqQlU3GkVsgTvcQjdEqmWFcMcPmpx9F7c/o4
7+8oa62Ln23VBeEs+TseYrlXOpDWvwir5uxSJw/P2en7sTzc6t+o8HCt/K2K7I6G
+E8I7aM3lD4FSYQU7VQfIdwpJR+SEfMRfCuLGwIDAQABAoIBACjHtvu8kEu1S2E7
cOe1JKGFQUilDGm0doU1VkY/2e4imWG7XLVdC6/EK2B9BkaENn3430tX1j+5ir/L
actUaqAGovhLcahvlWk9YX/Rje5WvSwdMg+f4tVboFl0zjo60lmWYbPXcuGWeALG
uHUJUoEGzbLvRXsqsfUGYDVVka1kWj12GzaZb9Mj/BpJUetXI5E9XazpZ8VGaXnP
PDrHFiZ89Nda9KpWFJ3s80VMhn4XmcpDkgE8OinLxtAo0YOE2dB+6x0G6pXpQ4+1
wqyvAS6AtpfTqhf5fenZVd6Nuuig28kNXWTHH8pYo1aLJS7by8yeXlJ17YX11zTT
lwQXLokCgYEA25jkNP+KRE0H8Lt2YVn7OzGRTVlRZQlP4CfqGpn5RzVkjYO+Nz7N
I3Q2Re+vsWwuseu+IlcIIfSzjDKugxdrgVtW5MxtS3jiYNz+S7z1638GxPre7H2y
SGKlcEuYixOAbMLLhSlY7QK95p2B44Xv/6m1nUS83v2zd3OCYqiMMBUCgYEA1VtP
WbI6Sl2S126/s1wuMjVRo40tPBWErbUzNoiJXqm9gNtX+7fsN9ei4mjR2pSA8Agm
RoMOIQ7u6F+EppS/u0P7RF6RDackieLJ6KHXuYhKBuHg70RO0fiYMMHP2pK4HG3d
nFspka4ghdVCqrabK+OQ2hk2gCH/ugNrlZISam8CgYEAs9c6zcyZx+XuItDj2kZ+
4bNudBI5/qzppYIKz05aQF8RwnOqTEQ6bFa4O/5XvM4ET+HpDOaJ2oU0phS7ptB/
UqkjOK3StISDoSBbglt2ay7UtG1gM/2dDHr9UkIW449NFcVSN/Psx+3+5+cGgPcn
3hF2kx3AMD9FwwXuOi4e4SECgYEAogo9U9RG7R/wXGoicih6dWuFW3/ncRCuufc6
tBoyqpCj+m+cfPMNFsRRUz9k0mujGao216rFWlorHJUe/B1RGPripORlqkbdO1Ph
ISt52dEm199JpK7uZg42GCG6qThxWDYg75VVFu12ie6UOW+CnmyxINOxTtODk1Tx
qqFMF0UCgYEAjnxNoweWHo3KXp8SfrXv2jFYRV3CTam1hE5hIMRp+7KZLiQPH9Lj
9UHlkrjlg8xOzzsHZfi3ExjX0mmwi1KK2CFIRPppD9oCLAEAgIppbFRm/MlS0fgD
91IcUYxLpRgitqjr8qEuAYEFHT22FBCRd2h7iCksqVILVtVgykaFB7U=
-----END RSA PRIVATE KEY-----

SCRIPT

scriptPUBKey = <<SCRIPT
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3BIlKVurhXeVK40UJzOvwZU5gtHGll6Bh0YjjuauSelIDn9Won3omda+Lvmnpf8xtlydlJ3RO4l0uaA4AooIexVem6qEawYkfIXqc4fcyn2pEqA1EfcXa6ciWSv2U1gY2iHssmcGk5j2N7GMmKQx4bGMdGbCPDZsfrwew3N5pHvGEGVfn7m2LKzicd0NxO3t31mM4+k1w6Xa+pCVTcaRWyBO9xCN0SqZYVwxw+anH0Xtz+jjv7yhrrYufbdUF4Sz5Ox5iuVc6kNa/CKvm7FInD8/Z6fuxPNzq36jwcK38rYrsjob4TwjtozeUPgVJhBTtVB8h3CklH5IR8xF8K4sb vagrant@loadbalancer
SCRIPT

scriptRSACaller = <<SCRIPT
echo "#{scriptPVTKey}" > /root/.ssh/id_rsa
sudo -H -u vagrant bash -c 'echo "#{scriptPVTKey}" > /home/vagrant/.ssh/id_rsa'
chmod 600 /root/.ssh/id_rsa
chmod 600 /home/vagrant/.ssh/id_rsa
echo "#{scriptPUBKey}" > /root/.ssh/id_rsa.pub
sudo -H -u vagrant bash -c 'echo "#{scriptPUBKey}" > /home/vagrant/.ssh/id_rsa.pub'
ssh-keyscan -t rsa "192.168.10.11" >> /root/.ssh/known_hosts
sudo -H -u vagrant bash -c 'ssh-keyscan -t rsa "192.168.10.11" >> /home/vagrant/.ssh/known_hosts'
SCRIPT

scriptRSACallee = <<SCRIPT
echo "#{scriptPUBKey}" >> /root/.ssh/authorized_keys
sudo -H -u vagrant bash -c 'echo "#{scriptPUBKey}" >> /home/vagrant/.ssh/authorized_keys'
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
    vlb.vm.provision "shell", :inline => scriptRSACaller
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
      app.vm.provision "shell", :inline => scriptRSACallee
      app.trigger.after :up do |trigger|
        trigger.run = { inline: 
          #"vagrant ssh loadbalancer -- cp .vagrant/machines/app-#{i+1}/virtualbox/private_key ~/.ssh/id_rsa"
          "vagrant ssh loadbalancer -- ssh-keyscan -t rsa 192.168.10.11 >> ~/.ssh/known_hosts"
        }
      end
    end
  end
end