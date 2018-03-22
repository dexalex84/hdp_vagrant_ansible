# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
# Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  # config.vm.box = "centos/7"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
#end

hard_conf = [
			 {
			     :type=>:name_node,:memory => 8192,:cpu => {:cpus => 2, :sockets => 1, :cores => 2, :threads => 1}
			 },
			 {
			     :type=>:data_node,:memory => 10240,:cpu => {:cpus => 4, :sockets => 2, :cores => 2, :threads => 1}
			     
			 },
			 {
			     :type=>:edge_node,:memory => 4094,:cpu => {:cpus => 2, :sockets => 1, :cores => 2, :threads => 1}
			 }
			]

NAME_NODE = "rnd-dwh-nn-001"
HDP_AMBARI_REPO = "http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.6.0.0/ambari.repo"
AMBARI_HOST_NAME = NAME_NODE

PLAYBOOK_PATH='provisioning/'
PLAYBOOK_NAME='hdp_ansible.yml'
			
nodes = [
		 {
		  :name => "name_node_1",
		  :host => "rnd-dwh-nn-001",
		  :type => :name_node,
		  :ntwk => {:ip => "192.168.0.120", :mac => "9E:1A:97:75:13:5C"}
		 },
		 {
		  :name => "data_node_1",
		  :host => "rnd-dwh-dn-001",
		  :type => :data_node,
		  :ntwk => {:ip => "192.168.0.122", :mac => "1C:0C:B7:9C:C9:D5"}
		 },		 
		 {
		  :name => "edge_node_1",
		  :host => "rnd-dwh-en-001",
		  :type => :edge_node,
		  :ntwk => {:ip => "192.168.0.121", :mac => "1C:0C:B7:9C:C9:D5"}
		 }
		]	
		
		
Vagrant.configure("2") do |config|    

   nodes.each do |node|
     
	 sel_hard_conf = hard_conf.select {|conf| conf[:type]==node[:type]}[0]
	 
	 config.vm.define node[:name] do |node_name|                                       
		node_name.vm.box = "centos/7" 
		node_name.vm.provider  :libvirt do |domain|
			domain.storage_pool_name = "vagrant"                                            
			domain.uri = 'qemu+unix:///system'                                              
			domain.host = node[:host]                                                  
			domain.memory = sel_hard_conf[:memory]                                                            
			domain.cpus = sel_hard_conf[:cpu][:cpus]                                                               
			domain.cputopology :sockets => sel_hard_conf[:cpu][:sockets], :cores => sel_hard_conf[:cpu][:cores], :threads => sel_hard_conf[:cpu][:threads]
                end
                
                node_name.vm.network :public_network,
                   :dev => "br0",
                   :type => "bridge",
                   :ip => node[:ntwk][:ip],
                   :mac => node[:ntwk][:mac]

	 end
   
  	config.vm.provision :hosts do |provisioner|
		provisioner.autoconfigure = false
		provisioner.add_localhost_hostnames = false
		nodes.each do |n|
			provisioner.add_host n[:ntwk][:ip], [ n[:host].to_s ]
		end
	end
    
   end
        config.vm.provision "ansible" do |ansible|
        
           ansible.extra_vars = {
              "hdp_ambari_repo" => HDP_AMBARI_REPO,
              "ambarihostname" => AMBARI_HOST_NAME 
           }

          ansible.playbook = "%s/%s" % [PLAYBOOK_PATH, PLAYBOOK_NAME]

        end

   

end 
