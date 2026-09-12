Vagrant.configure("2") do |config|
  config.vm.define "foreman" do |foreman|
    foreman.vm.box = "almalinux/9"
    foreman.vm.hostname = "foreman.example.local"
    foreman.vm.network "private_network", ip: "192.168.56.10"
    foreman.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
    end
  end

  config.vm.define "client" do |client|
    client.vm.box = "almalinux/9"
    client.vm.hostname = "client.example.local"
    client.vm.network "private_network", ip: "192.168.56.20"
    client.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
    end
  end
end
