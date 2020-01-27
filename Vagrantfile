# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.define :zfs do |zfs|
    zfs.vm.box = "debian/buster64"
    zfs.vm.hostname = "nfs"
    #zfs.vm.network :public_network,:bridge=>"enp2s0"
    zfs.vm.network :public_network,:bridge=>"wlp1s0"
  end

end
