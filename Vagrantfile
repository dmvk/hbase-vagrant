Vagrant.require_version ">= 1.8.0"

$nodes = 1;
$prefix = 'hbase'
$subnet = '172.17.8'
$master_memory = 512
$node_memory = 1024

$install_ansible = <<SCRIPT
ansible --version &> /dev/null || apt-get -y install software-properties-common
ansible --version &> /dev/null || apt-add-repository ppa:ansible/ansible
ansible --version &> /dev/null || apt-get -y update
ansible --version &> /dev/null || apt-get -y install ansible
SCRIPT

Vagrant.configure("2") do |config|

	config.vm.box = 'ubuntu/trusty64'

	if Vagrant.has_plugin?("vagrant-vbguest") then
		config.vbguest.auto_update = false
	end

	(1..$nodes).each do |i|
		config.vm.define vm_name = '%s-%02d' %  [$prefix, i] do |node|
			node.vm.hostname = vm_name
			node.vm.network :private_network, ip: "#{$subnet}.#{100 + i}"

			node.vm.provider :virtualbox do |vb|
				vb.cpus = 2
				vb.memory = $node_memory
			end
		end
	end

	config.vm.define vm_name = '%s-master' % $prefix do |node|
		node.vm.hostname = vm_name
		node.vm.network :private_network, ip: "#{$subnet}.100"

		node.vm.provider :virtualbox do |vb|
			vb.cpus = 2
			vb.memory = $master_memory
		end

		node.vm.provision 'shell', inline: $install_ansible

		# Patch for https://github.com/mitchellh/vagrant/issues/6793
		node.vm.provision "shell" do |s|
			s.inline = '[[ ! -f $1 ]] || grep -F -q "$2" $1 || sed -i "/__main__/a \\    $2" $1'
			s.args = ['/usr/bin/ansible-galaxy', "if sys.argv == ['/usr/bin/ansible-galaxy', '--help']: sys.argv.insert(1, 'info')"]
		end

		node.vm.provision :ansible_local do |ansible|
			ansible.playbook = 'playbook.yml'
			ansible.inventory_path = 'inventory'
			ansible.verbose = true
			ansible.sudo = true
			ansible.limit = 'all'
		end
	end
end
