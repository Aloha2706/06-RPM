# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
    config.vm.box = "centos/7"
    
    config.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 1
    end
  
    config.vm.define "rpm" do |rpm|
      rpm.vm.hostname = "rpm"
    end

  end
  