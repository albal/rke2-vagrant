ENV['VAGRANT_I_KNOW_WHAT_IM_DOING_PLEASE_BE_QUIET'] = 'true'
ENV['VAGRANT_NO_PARALLEL'] = 'no'
Vagrant.require_version ">= 2.2.17"

Vagrant.configure("2") do |config|
  config.vagrant.plugins = ["vagrant-reload", "vagrant-rke2"]

  config.vm.provider "virtualbox" do |v|
    v.cpus = 2    
    v.memory = 4096
    v.linked_clone = true
  end

  master_ip = "10.10.10.100"
  worker_start_subnet = "10.10.10"
  worker_start_octal = 100 # we always start at an index of 1, so first worker starts at .101
  distro = "ubuntu/jammy64"
  workers = 2

  config.vm.define "master" do |master|
    master.vm.box = distro
    master.vm.hostname = 'master'
    master.vm.network "private_network", ip: master_ip, netmask: "255.255.255.0"

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
  
  (1..workers).each do |i|

    config.vm.define "worker#{i}" do |worker|
      worker_ip = worker_start_subnet + ".#{worker_start_octal + i}"
      worker.vm.box = distro
      worker.vm.hostname = "worker-node#{i}"
      worker.vm.network "private_network", ip: "#{worker_ip}", netmask: "255.255.255.0"

      worker.vm.provision :rke2, run: "once" do |rke2|
        rke2.env = %w[INSTALL_RKE2_CHANNEL=stable INSTALL_RKE2_TYPE=agent]
        rke2.config_mode = '0644' # side-step https://github.com/k3s-io/k3s/issues/4321
        rke2.config = <<~YAML
          write-kubeconfig-mode: 0644
          node-external-ip: #{worker_ip}
          node-ip: #{worker_ip}
          server: https://#{master_ip}:9345
          token: vagrant-rke2
        YAML
      end
    end

  end
   
end