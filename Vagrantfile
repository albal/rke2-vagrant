ENV['VAGRANT_I_KNOW_WHAT_IM_DOING_PLEASE_BE_QUIET'] = 'false'
ENV['VAGRANT_NO_PARALLEL'] = 'no'
Vagrant.require_version ">= 2.2.17"

Vagrant.configure("2") do |config|
  config.vagrant.plugins = ["vagrant-reload", "vagrant-rke2"]

  config.vm.provider "virtualbox" do |v|
    v.cpus = 4    
    v.memory = 8192
    v.linked_clone = false
  end

  master_ip = "10.10.10.100"
  worker1_ip = "10.10.10.101"
  worker2_ip = "10.10.10.102"
  distro = "generic/ubuntu2004"

  config.vm.define "master" do |master|
    master.vm.box = distro
    master.vm.hostname = 'master'
    master.vm.network "private_network", ip: master_ip, netmask: "255.255.255.0"
    #master.vm.provision "Set DNS", type: "shell", inline: "systemd-resolve --set-dns=8.8.8.8 --interface=eth0"

    master.vm.provision :rke2, run: "once" do |rke2|
      rke2.env = %w[INSTALL_RKE2_CHANNEL=stable INSTALL_RKE2_TYPE=server]
      rke2.config_mode = '0644' # side-step https://github.com/k3s-io/k3s/issues/4321
       # note that write-kubeconfig-mode needs to be a string
      rke2.config = <<~YAML
        write-kubeconfig-mode: '0644'
        node-external-ip: #{master_ip}
        node-ip: #{master_ip}

        token: vagrant-rke2
        cni: calico
      YAML
    end
  end
  
  config.vm.define "worker1" do |worker1|
    worker1.vm.box = distro
    worker1.vm.hostname = 'worker1'
    worker1.vm.network "private_network", ip: worker1_ip, netmask: "255.255.255.0"
    #worker1.vm.provision "Set DNS", type: "shell", inline: "systemd-resolve --set-dns=8.8.8.8 --interface=eth0"

    worker1.vm.provision :rke2, run: "once" do |rke2|
      rke2.env = %w[INSTALL_RKE2_CHANNEL=stable INSTALL_RKE2_TYPE=agent]
      rke2.config_mode = '0644' # side-step https://github.com/k3s-io/k3s/issues/4321
      rke2.config = <<~YAML
        write-kubeconfig-mode: 0644
        node-external-ip: #{worker1_ip}
        node-ip: #{worker1_ip}
        
        master: https://#{master_ip}:9345
        token: vagrant-rke2
      YAML
    end
  end
  
  config.vm.define "worker2" do |worker2|
    worker2.vm.box = distro
    worker2.vm.hostname = 'worker2'
    worker2.vm.network "private_network", ip: worker2_ip, netmask: "255.255.255.0"
    #worker2.vm.provision "Set DNS", type: "shell", inline: "systemd-resolve --set-dns=8.8.8.8 --interface=eth0"

    worker2.vm.provision :rke2, run: "once" do |rke2|
      rke2.env = %w[INSTALL_RKE2_CHANNEL=stable INSTALL_RKE2_TYPE=agent]
      rke2.config_mode = '0644' # side-step https://github.com/k3s-io/k3s/issues/4321
      rke2.config = <<~YAML
        write-kubeconfig-mode: 0644
        node-external-ip: #{worker2_ip}
        node-ip: #{worker2_ip}
        
        master: https://#{master_ip}:9345
        token: vagrant-rke2
      YAML
    end
  end

end