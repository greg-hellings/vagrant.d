# -*- mode: ruby -*-
# vi: ft=ruby:noexpandtab:ts=4

# Required for reading clouds.yaml config file
require 'yaml'

# This is the basic entry-point method here. You can use this
# to just create vms ad nauseum in your Vagrantfile. From within
# your main Vagrantfile configuration block, just call to this
# method to create as many VMs as you want. So your Vagrantfile
# can look as simple as
# Vagrant.configure('2') do config
#   vm(config, "myhost")
# end
def vm(config, name, base_box='generic/fedora35')
	config.vm.define name do |node|
		node.vm.provider :libvirt do |libvirt, override|
			local_setup(override, base_box)
			libvirt.memory = 4096
			libvirt.cpus = 2
		end
		node.vm.provider :virtualbox do |virtualbox, override|
			local_setup(override, base_box)
		end
		node.vm.hostname = name + ".box"
		node.vm.synced_folder ".", "/vagrant", disabled: true
		node.vm.provider :openstack do |os, override|
			os.image = get_image(base_box)
			override.ssh.username = get_username(base_box)
			os.server_name = 'pet-' + ENV['USER'] + '-' + name
		end
		yield node if block_given?
	end
end

# Use this method to add Ansible integration to your Vagrantfile.
# After creating the machine by the above method, you can pass it
# into here to have Ansible run as the provisioner on guest
# creation. You can do something like
# vm(config, "myhost") do node
#   ansible(node)
# end
# in order to run the playbook.
#
# The argument "book" is the path, relative to the main Vagrantfile,
# to the playbook in your repository that ought to be run. If there
# are muliple playbooks that need to be run, in succession, you
# should be able to call this method multiple times
def ansible(config, book=ENV['HOME'] + '/src/ansible_collections/devroles/system/playbooks/workstation.yml')
	config.vm.provision :ansible do |ansible|
		ansible.limit = "all"
		ansible.verbose = "-v"
		ansible.playbook = book
		ansible.groups = {
		  	"admin" => config.vm.defined_vms.keys
		}
		yield ansible if block_given?
	end
end

# A few methods that are used as helpers in the above work
$ip = 2

def local_setup(node, base_box)
	node.vm.box = base_box
	node.vm.network "private_network", ip: "192.168.8.#{$ip}", netmask: "255.255.255.0", libvirt__guest_ipv6: 'yes'
	$ip += 1
	node.vm.synced_folder "../..", "/home/vagrant/dev", type: "sshfs", sshfs_opts_append: "-o idmap=user"
end

def get_username(base_box)
	if base_box.start_with?('centos')
		return 'centos'
	elsif base_box.start_with?('rhel')
		return 'root'
	else
		return 'fedora'
	end
end

def get_image(base_box)
	if base_box == 'centos/7'
		return 'CentOS-7-x86_64-GenericCloud-released-latest'
	elsif base_box == 'centos/6'
		return 'CentOS-6-x86_64-GenericCloud-released-latest'
	elsif base_box == 'fedora/25-cloud-base'
		return 'Fedora-Cloud-Base-25-compose-latest'
	elsif base_box == 'fedora/24-cloud-base'
		return 'Fedora-Cloud-Base-24-compose-latest'
	elsif base_box == 'fedora/27-cloud-base'
		return 'Fedora-Cloud-Base-27-1.6'
	elsif base_box == 'fedora/28-cloud-base'
		return 'Fedora-Cloud-Base-28-compose-latest'
	elsif base_box == 'fedora/29-cloud-base'
		return 'Fedora 29'
	elsif base_box == 'generic/fedora35'
		return 'Fedora-Cloud-Base-35'
	elsif base_box == 'rhel7.2'
		return 'rhel-7.2-server-x86_64-updated'
	elsif base_box == 'rhel7.3'
		return 'rhel-7.3-server-x86_64-updated'
	elsif base_box == 'rhel7.4'
		return 'rhel-7.4-server-x86_64-updated'
	elsif base_box == 'generic/rhel8'
		return 'rhel-8.0-x86_64-latest'
	elsif base_box == 'rhel6'
		return 'rhel-6.9-server-x86_64-updated'
	end
end

# This code allows you to use the OpenStack provider to run your VMs while
# not configuring anything more than the normal OS_* named variables that
# the os-client-config Python packages also understand. That way, if your
# host machine is already configured for easy access to your OpenStack instance,
# then you can just run "vagrant --provider openstack up" and this will auto
# configure auth
Vagrant.configure("2") do |config|
	config.vm.provider :openstack do |os, override|
		clouds = YAML.load_file(ENV['HOME'] + '/.config/openstack/clouds.yaml')
		cloud = clouds['clouds']['default']
		if ENV.key?('OS_CLOUD')
			cloud = clouds['clouds'][ENV['OS_CLOUD']]
		end
		os.openstack_auth_url = cloud['auth']['auth_url']
		os.username = cloud['auth']['username']
		os.password = cloud['auth']['password']
		os.flavor = cloud['nova_flavor']
		os.networks = cloud['networks'].map{ |net| net['name'] }
		os.server_create_timeout = 900
		if cloud.key?('floating_network')
			os.floating_ip_pool = cloud['floating_network']
		end
		if cloud['auth'].key?('user_domain_name')
			os.user_domain_name = cloud['auth']['user_domain_name']
		end
		if cloud['auth'].key?('project_name')
			os.tenant_name = cloud['auth']['project_name']
			os.project_name = cloud['auth']['project_name']
			os.project_domain_name = cloud['auth']['project_domain_name']
		end
		if cloud.key?('identity_api_version')
			os.identity_api_version = cloud['identity_api_version'].to_s
		end
	end
end
