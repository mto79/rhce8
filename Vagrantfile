# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV["LC_ALL"] = "en_US.UTF-8"

user = ENV["RH_SUBSCRIPTION_MANAGER_USER"] # SET in environment 
password = ENV["RH_SUBSCRIPTION_MANAGER_PASS"] # SET in environment
if !user or !password
  puts 'Required environment variables not found. Please set RH_SUBSCRIPTION_MANAGER_USER and RH_SUBSCRIPTION_MANAGER_PW'
  abort
end

$register_script = %{
if ! subscription-manager status; then
  sudo subscription-manager register --username=#{user} --password=#{password} --auto-attach
fi
}

$unregister_script = %{
if subscription-manager status; then
  sudo subscription-manager unregister
fi
}

$setup_script = %{
if subscription-manager status; then
   sudo yum install -y python3 python3-libselinux
fi
}

Vagrant.configure("2") do |config|
    
  def createNode(config, name, box, version, memory, cpus, ip)
    config.ssh.insert_key = false 
    
    config.vm.define "#{name}" do |node|
      node.vm.box = "#{box}"
      node.vm.hostname = "#{name}.local"
      
      node.vm.network :private_network, 
       :ip => "#{ip}",
       :libvirt__network_name => "default",
       :autostart => true  

      # Libvirt provider
      node.vm.provider :libvirt do |lv|
        lv.driver = "kvm" 
        lv.memory = "#{memory}"
        lv.cpus = "#{cpus}"
        # lv.storage :file, :size => '40G', :type => 'raw'
        lv.nested = true
        lv.disk_driver :cache => "none"
        lv.storage_pool_name = "default"
        lv.sound_type = "ich6"
        lv.graphics_type = "spice"
        lv.video_type = "qxl"
        lv.channel :type => "unix", :target_name => "org.qemu.guest_agent.0", :target_type => "virtio"
        lv.channel :type => "spicevmc", :target_name => "com.redhat.spice.0", :target_type => "virtio"
        lv.default_prefix = ""  
      end 
      
      # Virtualbox provider
      # node.vm.provider :virtualbox do |vb|
      #  vb.memory = "#{memory}"
      #  vb.cpus = "#{cpus}"
      # end
      
      node.vm.provision "shell", inline: $register_script
      node.vm.provision "shell", inline: $setup_script
      
      node.trigger.before :destroy do |trigger|
        trigger.name = "Before Destroy trigger"
        trigger.info = "Unregistering this VM from RedHat Subscription Manager..."
        trigger.warn = "If this fails, unregister VMs manually at https://access.redhat.com/management/subscriptions"
        trigger.run_remote = {inline: $unregister_script}
        trigger.on_error = :continue
      end # trigger.before :destroy    

    end
  
  end   
  
  createNode(config, "control", "generic/rhel8", "3.5.0", "2048", "2", "192.168.122.10")
  createNode(config, "ansible1", "generic/rhel8", "3.5.0", "2048", "2", "192.168.122.11")
  createNode(config, "ansible2", "generic/rhel8", "3.5.0", "2048", "2", "192.168.122.12")
  createNode(config, "ansible3", "generic/rhel8", "3.5.0", "2048", "2", "192.168.122.13")
  createNode(config, "ansible4", "generic/rhel8", "3.5.0", "2048", "2", "192.168.122.14")

end 