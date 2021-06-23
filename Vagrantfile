# -*- mode: ruby -*-
# vi: set ft=ruby:

Vagrant.configure("2") do |config|

  servers=[
    {
      :hostname => "k8s-master",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.88.210",
      :ssh_port => '2210',
    },
    {
      :hostname => "k8s-node1",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.88.211",
      :ssh_port => '2211'
    },
    {
      :hostname => "k8s-node2",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.88.212",
      :ssh_port => '2212'
    },
    {
      :hostname => "k8s-node3",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.88.213",
      :ssh_port => '2213'
    }
  ]

  servers.each do |machine|

    config.vm.define machine[:hostname] do |node|
      node.vm.box = machine[:box]
      node.vm.hostname = machine[:hostname]
      
      node.vm.network :public_network, ip: machine[:ip]
      node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"

      node.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", 4096, "--cpus", 3, "--cpuexecutioncap", 60]
        v.customize ["modifyvm", :id, "--name", machine[:hostname]]
      end
    end
  end

#  config.vbguest.installer_options = { allow_kernel_upgrade: true }
  config.vbguest.auto_update = false
  config.vbguest.no_remote = true

  config.vm.provision "file", source: "ssh_key/id_rsa", destination: "/home/vagrant/.ssh/"
  config.vm.provision "file", source: "ssh_key/id_rsa.pub", destination: "/home/vagrant/.ssh/"

  config.vm.provision "shell",
    inline: "set -x && \
             sed -i 's/^#.*StrictHostKeyChecking ask/    StrictHostKeyChecking no/' /etc/ssh/ssh_config && \
             if [[ $(cat /vagrant/ssh_key/id_rsa.pub | xargs -I {} grep {} ~/.ssh/authorized_keys | wc -l) -eq 0 ]]; then cat /vagrant/ssh_key/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys; fi && \
             chmod 644 /home/vagrant/.ssh/id_rsa.pub && \
             chmod 600 /home/vagrant/.ssh/id_rsa && \
            "

  config.vm.define "k8s-master" do |node1|
    node1.vm.provision "shell",
      inline: "apt-get update && apt install python3-pip -y && \
               curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
               python3 get-pip.py --force-reinstall && \
               pip3 install ansible && \
               echo 'if [[ -d ~/.local/bin ]];then export PATH=~/.local/bin:$PATH;fi' | tee -a ~/.bashrc && source ~/.bashrc && \
               cp -r /vagrant/provisioning /home/vagrant/ && \
               chmod o-w /home/vagrant/provisioning && \
               chown -R vagrant:vagrant /home/vagrant/provisioning && \
               if [[ ! -e /vagrant/ssh_key/id_rsa ]]; then ssh-keygen -b 2048 -t rsa -f /vagrant/ssh_key/id_rsa -q -N ""; fi 
              "
  end
end
