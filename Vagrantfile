# -*- mode: ruby -*-
# # vi: set ft=ruby :

Vagrant.configure(2) do |config|

  (1..2).each do |i|
    config.vm.define "k8s#{i}" do |s|
      s.ssh.forward_agent = true
      s.vm.box = "ubuntu/xenial64"
      s.vm.hostname = "k8s#{i}"
      s.vm.provision :shell, path: "scripts/bootstrap_ansible.sh"
      if i == 1
        s.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-master.yml -c local"
      else
        s.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-worker.yml -c local"
      end
      s.vm.network "private_network", ip: "172.42.42.#{i+10}", netmask: "255.255.255.0",
                   auto_config: true,
                   virtualbox__intnet: "k8s-net"
      s.vm.provider "virtualbox" do |v|
        v.name = "k8s#{i}"
        v.memory = 4096
        v.gui = false
      end
      if i != 1
        # Add a route back to the kubernetes API service
        s.vm.provision :shell,
        run: "always",
        inline: "echo Setting Cluster Route; clustip=$(kubectl --kubeconfig=admin.conf get svc -o json | JSONPath.sh '$.items[?(@.metadata.name=kubernetes)]..clusterIP' -b); node=$(kubectl --kubeconfig=admin.conf get endpoints -o json | JSONPath.sh '$.items[?(@.metadata.name=kubernetes)]..ip' -b); ip route add $clustip/32 via $node || true"
      end
    end
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

end
