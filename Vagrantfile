Vagrant.configure("2") do |config|
  config.vm.box = "centos/8"
  config.vm.hostname = "larabox8"
  config.vm.network "public_network", ip: "192.168.0.100", netmask: "255.255.255.0", gateway: "192.168.0.1"

  config.vm.provider "virtualbox" do |vb|
     vb.cpus = 1
     vb.memory = "512"
  end
end
