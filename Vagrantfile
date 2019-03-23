# -*- mode: ruby -*-
# vi: set ft=ruby :


# https://github.com/hashicorp/vagrant/issues/1874#issuecomment-165904024
# not using 'vagrant-vbguest' vagrant plugin because now using bento images which has vbguestadditions preinstalled.
def ensure_plugins(plugins)
  logger = Vagrant::UI::Colored.new
  result = false
  plugins.each do |p|
    pm = Vagrant::Plugin::Manager.new(
      Vagrant::Plugin::Manager.user_plugins_file
    )
    plugin_hash = pm.installed_plugins
    next if plugin_hash.has_key?(p)
    result = true
    logger.warn("Installing plugin #{p}")
    pm.install_plugin(p)
  end
  if result
    logger.warn('Re-run vagrant up now that plugins are installed')
    exit
  end
end

required_plugins = ['vagrant-hosts', 'vagrant-share', 'vagrant-vbox-snapshot', 'vagrant-host-shell', 'vagrant-reload']
ensure_plugins required_plugins



Vagrant.configure(2) do |config|

  config.vm.define "nfs_storage" do |nfs_storage_config|
    nfs_storage_config.vm.box = "bento/centos-7.6"
    nfs_storage_config.vm.hostname = "nfs-storage.local"
    # https://www.vagrantup.com/docs/virtualbox/networking.html
    nfs_storage_config.vm.network "private_network", ip: "10.3.5.109", :netmask => "255.255.255.0", virtualbox__intnet: "intnet2"

    nfs_storage_config.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.memory = "1024"
      vb.cpus = 2
      vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      vb.name = "centos7_nfs_storage"
    end

    nfs_storage_config.vm.provision "shell", path: "scripts/install-rpms.sh", privileged: true
    nfs_storage_config.vm.provision "shell", path: "scripts/nfs_server_setup.sh", privileged: true
  end




  config.vm.define "kube_master" do |kube_master|
    kube_master.vm.box = "bento/ubuntu-16.04"
    kube_master.vm.hostname = "kube-master.example.com"
    # https://www.vagrantup.com/docs/virtualbox/networking.html
    kube_master.vm.network "private_network", ip: "10.3.5.110", :netmask => "255.255.255.0", virtualbox__intnet: "intnet2"
    # you can view this ip by running:   hostname -I
    # the following is needed for accessing the kubernetes dashboard, but can't get it working yet. 
    kube_master.vm.network "forwarded_port", guest: 31000, host: 8500, protocol: 'tcp'



    kube_master.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.memory = "1024"
      vb.cpus = 2
      vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      vb.name = "centos7_kube_master_nfs_demo"
    end

    kube_master.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = '2.0'
      ansible.extra_vars = {
        eth1_ip_address: "10.3.5.110"
      }
      ansible.playbook = "setup-kube-master.yml"
    end


  end

  (1..1).each do |i|
    config.vm.define "kube_worker#{i}" do |kube_worker|
      kube_worker.vm.box = "bento/ubuntu-16.04"
      kube_worker.vm.hostname = "kube-worker#{i}.example.com"
      kube_worker.vm.network "private_network", ip: "10.3.5.11#{i}", :netmask => "255.255.255.0", virtualbox__intnet: "intnet2"

      kube_worker.vm.provider "virtualbox" do |vb|
        vb.gui = false
        vb.memory = "1024"
        vb.cpus = 2
        vb.name = "centos7_kube_worker#{i}_nfs_demo"
      end

      kube_worker.vm.provision "ansible" do |ansible|
        ansible.compatibility_mode = '2.0'
        ansible.extra_vars = {
          eth1_ip_address: "10.3.5.11#{i}"
        }
        ansible.playbook = "setup-kube-worker.yml"
      end

    end
  end

#  config.vm.provision :hosts do |provisioner|
#    provisioner.add_host '10.3.5.110', ['kube-master.example.com']
#    provisioner.add_host '10.3.5.111', ['kube-worker1.example.com']
#  end




end