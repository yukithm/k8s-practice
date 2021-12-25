# -*- mode: ruby -*-
# vi: set ft=ruby :

# number of worker nodes
NUM_OF_WORKERS = 2

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

  (1..NUM_OF_WORKERS).each do |worker_id|
    config.vm.define "worker#{worker_id}" do |worker|
      worker.vm.hostname = "worker#{worker_id}"
      worker.vm.network "private_network", ip: "192.168.56.#{170 + worker_id}"
      worker.vm.provider "virtualbox" do |vb|
        vb.gui = false
        vb.cpus = 1
        vb.memory = 2048
      end
    end
  end

  config.vm.provision "ansible" do |ansible|
    ansible.verbose = "v"
    ansible.config_file = "./ansible/ansible.cfg"
    ansible.playbook = "./ansible/site.yml"
    ansible.groups = {
      "control_plane" => ["control-plane"],
      "workers" => (1..NUM_OF_WORKERS).map {|id| "worker#{id}"}
    }
  end
end
