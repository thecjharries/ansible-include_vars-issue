# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  config.vm.provision "file", source: "./provisioning", destination: "/tmp/provisioning"

  config.vm.provision :shell do |sh|
    sh.path = "provision.sh"
    sh.args = ["vagrant"]
    sh.privileged = true
  end
end
