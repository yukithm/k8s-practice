# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vagrant.plugins = ["vagrant-vbguest"]

  config.vm.box = "ubuntu/focal64"

  config.vm.define "control-plane" do |master|
    master.vm.hostname = "control-plane"
    master.vm.network "private_network", ip: "192.168.56.161"
    master.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.cpus = 2
      vb.memory = 2048
    end
  end

  config.vm.provision "ansible" do |ansible|
    ansible.verbose = "v"
    ansible.config_file = "./ansible/ansible.cfg"
    ansible.playbook = "./ansible/site.yml"
    ansible.groups = {
      "control_plane" => ["control-plane"],
      "workers" => ["worker1", "worker2"]
    }
  end
end
