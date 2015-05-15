require 'json'

VAGRANTFILE_API_VERSION = '2'

# Configuration file
profile_file = 'profile.json'

# Parse the file as JSON
profile = JSON.parse(IO.read(profile_file), opts={symbolize_names: true})

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Try to enable vagrant-cachier
  if Vagrant.has_plugin?('vagrant-cachier')
    # Configure cached packages to be shared between instances of the same base box.
    config.cache.scope= :box
  end

  config.vm.box = profile[:box_name]
  # Put up the box somewhere?
  #config.vm.box_url = profile[:box_url]

  # Extract memory and cpu usage
  config.vm.provider :virtualbox do |vbox|
    vbox.customize(['modifyvm', :id, '--memory', profile[:memory]])
    vbox.customize(['modifyvm', :id, '--cpus', profile[:cpus]])
  end

  # The domain for the nodes
  domain = profile[:domain]

  # Each defined node
  profile[:nodes].each do |node|
    hostname = "#{node[:hostname]}.#{domain}"
    config.vm.define node[:hostname] do |node_config|
      # Setup networking
      node_config.vm.hostname = hostname
      node_config.vm.network :private_network, ip: node[:ip]
      node_config.ssh.forward_agent = true

      # Set up host<->guest directories
      sync_folder = profile[:sync_folder]
      unless sync_folder.nil?
        node_config.vm.synced_folder sync_folder[:src], sync_folder[:dest], owner: sync_folder[:owner], group: sync_folder[:group]
      end
    end
  end
end
