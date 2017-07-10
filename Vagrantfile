# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
	config.vm.box = "base"
	config.vm.synced_folder ".", "/ansible"
	config.vm.provider "virtualbox" do |vb|
		vb.memory = "1024"
	end
	config.vm.box = "centos/7"

	config.vm.define "master-4" do |node|
		node.vm.network "private_network", ip: "192.168.33.4"
		node.vm.network "forwarded_port", guest: 5432, host: 5432
		node.vm.provision "ansible" do |ansible|
			provisioner(ansible)
		end
	end


	(5..6).each do |i| 
		config.vm.define "slaves-#{i}" do |node|
			node.vm.network "private_network", ip: "192.168.33.#{i}"
			node.vm.provision "ansible" do |ansible|
				provisioner(ansible)
			end
		end
	end
end

def provisioner(ansible)
	ansible.playbook = "role.yml"
	ansible.inventory_path = "vagrant_ansible_inventory"

	ansible.groups = {
		"streaming-master" => ["master-4"],
		"streaming-slaves"=> ["slaves-5", "slaves-6"],
		"master" => ["master-4"],
		"slaves"=> ["slaves-5", "slaves-6"],
	}

	ansible.host_vars = {
		"master-4": {
			"private_ip": "192.168.33.4",
		},
		"slaves-5": {
			"private_ip": "192.168.33.5",
		},
		"slaves-6": {
			"private_ip": "192.168.33.6",
		},
	}

	ansible.extra_vars = {
		postgresql_conf_default_guc: [
			{ regexp: "^#?max_connections = .*$", guc: "max_connections = 64" },
			{ regexp: "^#?listen_addresses = .*$", guc: "listen_addresses = '*'" },
			{ regexp: "^#?wal_level = .*$", guc: "wal_level = {{'hot_standby'}}" },
			{ regexp: "^#?max_wal_senders = .*$", guc: "max_wal_senders = {{ groups['slaves']|length * 5 }}" },
			{ regexp: "^#?wal_keep_segments = .*$", guc: "wal_keep_segments = {{ 256 }}" },
			{ regexp: "^#?hot_standby = .*$", guc: "hot_standby = {{ 'on' }}" },
		],
		postgresql_streaming_user: {
			name: "replica",
			pass: "replica",
		},
		postgresql_extensions: [
			"hstore",
			"pg_stat_statements",
			"pgcrypto",
			"uuid-ossp",
		]
	}
end

