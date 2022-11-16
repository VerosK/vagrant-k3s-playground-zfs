# -*- mode: ruby -*-
# vi: set ft=ruby :

KUBECTL_PORT = 6443
UBUNTU = "bento/ubuntu-22.04"

VIRTUAL_MACHINES = {
  :zoo => {
    :ip    => '192.168.49.41',
    :offset => 0,
    :box   => UBUNTU,
  },
}
APISERVERS = [ :master1 ]


Vagrant.configure("2") do |config|

  VIRTUAL_MACHINES.each do |name,cfg|
	  config.vm.define name do |machine|
		  machine.vm.hostname = name
		  machine.vm.box = cfg[:box]
		  machine.vm.synced_folder ".", "/vagrant", type: "nfs", disabled: true

		  machine.vm.network "private_network", ip: cfg[:ip]

		  machine.vm.box_check_update = false
		  machine.vm.network "forwarded_port", guest: 6443, host: KUBECTL_PORT + cfg[:offset]


		  machine.vm.provider "virtualbox" do |vb|
			# be faster
			vb.linked_clone = true
			vb.memory = 4096
			vb.cpus = 2

			disk_name = "#{name}-pool.vdi"
			vb.customize ['createhd', '--filename', disk_name, '--size', 10 * 1024]
			vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', disk_name]
		  end


  	  	 machine.vm.provision :ansible do |ansible|
			ansible.playbook = 'setup-hosts.yml'

			ansible.become = true
			ansible.verbose = "v"
			ansible.raw_arguments = ['--diff']

			if ENV['ANSIBLE_TAGS'] then ansible.tags = ENV['ANSIBLE_TAGS']; end

			ansible.groups = {
				"vagrant" => VIRTUAL_MACHINES.keys,
				"vagrant-apiservers" => APISERVERS,
			}
			ansible.extra_vars = {
				ansible_python_interpreter: 'python3',
			}
		  end


	 end
  end

end

