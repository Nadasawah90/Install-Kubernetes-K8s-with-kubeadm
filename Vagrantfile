Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true 
  config.hostmanager.manage_host = true
  
###LB##
### LB vm  ####
  config.vm.define "lb" do |lb|
    lb.vm.box = "eurolinux-vagrant/centos-stream-9"
    lb.vm.box_version = "9.0.43"
    lb.vm.hostname = "lb"
    lb.vm.network "private_network", ip: "192.168.56.14"
	lb.vm.disk :disk, size: "15GB", primary: true
    lb.vm.provider "virtualbox" do |vb|
     vb.memory = "1024"
	 vb.cpus = 1
	 
    
	end
 end
  
### master1 vm  ####
  config.vm.define "master01" do |master01|
    master01.vm.box = "eurolinux-vagrant/centos-stream-9"
    master01.vm.box_version = "9.0.43"
    master01.vm.hostname = "master01"
    master01.vm.network "private_network", ip: "192.168.56.15"
	master01.vm.disk :disk, size: "15GB", primary: true
    master01.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
	 vb.cpus = 2
	 
    
	end
 end
 ### master2 vm  ####
  config.vm.define "master02" do |master02|
    master02.vm.box = "eurolinux-vagrant/centos-stream-9"
    master02.vm.box_version = "9.0.43"
    master02.vm.hostname = "master02"
    master02.vm.network "private_network", ip: "192.168.56.16"
	master02.vm.disk :disk, size: "15GB", primary: true
    master02.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
	 vb.cpus = 2
	 
    
	end
 end
 ### master3 vm  ####
  config.vm.define "master03" do |master03|
    master03.vm.box = "eurolinux-vagrant/centos-stream-9"
    master03.vm.box_version = "9.0.43"
    master03.vm.hostname = "master03"
    master03.vm.network "private_network", ip: "192.168.56.17"
	master03.vm.disk :disk, size: "15GB", primary: true
    master03.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
	 vb.cpus = 2
	 
    
	end
 end
### worker01 vm  ####
  config.vm.define "worker01" do |worker01|
    worker01.vm.box = "eurolinux-vagrant/centos-stream-9"
    worker01.vm.box_version = "9.0.43"
    worker01.vm.hostname = "worker01"
    worker01.vm.network "private_network", ip: "192.168.56.18"
	worker01.vm.disk :disk, size: "15GB", primary: true
    worker01.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
	 vb.cpus = 2
	 
	end
 end


### worker02 vm  ####
  config.vm.define "worker02" do |worker02|
    worker02.vm.box = "eurolinux-vagrant/centos-stream-9"
    worker02.vm.box_version = "9.0.43"
    worker02.vm.hostname = "worker02"
    worker02.vm.network "private_network", ip: "192.168.56.19"
	worker02.vm.disk :disk, size: "15GB", primary: true
    worker02.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
	 vb.cpus = 2


end
  end
 end

