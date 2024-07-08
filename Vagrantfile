# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

# GET CONFIGURATION FOR VMS
ENVIROMENT = YAML.load_file('enviroment.yml')
VMS = ENVIROMENT['vms']

# Convert configuration to list of VMs if necessary
if not VMS.is_a?(Array)
  VMS = [VMS]
end
# puts "VMS: #{VMS}" # REMOVE: print

def convert_to_array(data)
  if not data.is_a?(Array)
    data = [data]
  end 
  return data
end


def default_settings(vmdata)
  vm_config = {}
  default_config = {
    "hostname"          => vmdata["name"],
    "distro"            => "debian/bookworm64",
    "primary"           => false,
    "guest_additions"   => true,
    "ram"               => 2,
    "cpu"               => 2,
  }

  default_config.each do |key, default_value|
    if vmdata.key?(key)
      vm_config[key] = vmdata[key]
    else
      vm_config[key] = default_value
    end
  end
  return vmdata.merge(vm_config)
end


def synced_folder(vm, config)
  # puts "synced_folder config: #{config}" # REMOVE: print
  # Default folder configuration
  default = config.key?("default_disabled") ? config["default_disabled"] : false
  vm.synced_folder "./data", "/vagrant", disabled: default

  # Configure extra folders
  if config.fetch("extra", nil)
    sync_folders = convert_to_array(config["extra"])
    
    sync_folders.each do |sync_folder|
      type = "rsync"
      target = sync_folder["target"] # TODO: Mandatory
      dest = sync_folder["dest"] # TODO: Mandatory
      disabled = sync_folder["disabled"] ? sync_folder["disabled"] : false

      # Configure sync with type rsync
      vm.synced_folder target, dest, type: type, disabled: disabled
    end
  end
end


def disk(vm, config)
  # puts "disk config: #{config}" # REMOVE: print
  # Configure primary disk
  if config.fetch("primary", nil)
    diskcfg = config["primary"]
    diskname = diskcfg["name"] ? diskcfg["name"] : "primary"
    disksize = diskcfg["size"] ? diskcfg["size"] : "20GB"
    vm.disk :disk, size: disksize, name: diskname, primary: true
  end

  # Configure extra disks
  if config.fetch("extra", nil)
    extra_disks = convert_to_array(disk_cfg["extra"])
    extra_disks.each_with_index do |diskcfg, diskid|
      disksize = diskcfg["size"] ? diskcfg["size"] : "20GB"
      diskname = diskcfg["name"] ? diskcfg["name"] : "extradisk-#{diskid}"
      vm.disk :disk, size: disksize, name: diskname
    end
  end
end


def network(vm, vmid, config)
  # puts "network config: #{config}" # REMOVE: print
  # Configure public network
  if config.fetch("public", nil)
    networkcfg = config["public"]
    # puts "networkcfg: #{networkcfg}" # REMOVE: print
    network_type = networkcfg.fetch("type", nil)
    network_config = networkcfg["config"] ? networkcfg["config"] : {}

    # Configure public network as DHCP
    if network_type.downcase == "dhcp"
      if network_config.key?("use_dhcp_assigned_default_route")
        dhcp_route = network_config.fetch("use_dhcp_assigned_default_route")
        # puts "network dhcp 1: #{config}" # REMOVE: print
        vm.network "public_network",
          use_dhcp_assigned_default_route: dhcp_route
      else
        # puts "network dhcp 2: #{config}" # REMOVE: print
        vm.network "public_network"
      end
    
    # Configure public network as StaticIP
    elsif network_type.downcase == "staticip"
      if network_config.key?("ip")
        ip = network_config["ip"]
        vm.network "public_network", ip: ip
      else
        errormsg = "Campo vms[#{vmid}].network.public.config.ip mandatorio"
        raise KeyError.new(errormsg)
      end
    # Configure public network as Bridge
    elsif network_type.downcase == "bridge"
      if network_config.fetch("interfaces", nil)
        interfaces = network_config["interfaces"]
        if not interfaces.is_a?(Array)
          interfaces = [interfaces]
        end
        vm.network "public_network", bridge: interfaces
      else
        errormsg = "Campo vms[#{vmid}].network.public.config.interfaces mandatorio"
        raise KeyError.new(errormsg)
      end
    # Configure public network manually
    elsif network_type.downcase == "manual"
      config.vm.network "public_network", auto_config: false
    else
      errormsg = "Campo vms[#{vmid}].network.public.type:#{network_type} incorreto"
      raise KeyError.new(errormsg)
    end
  end

  # Configure private network
  if config.fetch("private", nil)
    networkcfgs = config["private"]
    # puts "networkcfgs: #{networkcfgs}" # REMOVE: print
    if not networkcfgs.is_a?(Array)
      networkcfgs = [networkcfgs]
    end
    networkcfgs.each_with_index do |networkcfg, privateid|
      network_type = networkcfg.fetch("type", nil)
      network_config = networkcfg["config"] ? networkcfg["config"] : {}
      
      # Configure private network as DHCP
      if network_type.downcase == "dhcp"
        # puts "network private dhcp: #{networkcfg}" # REMOVE: print
        vm.network "private_network", type: "dhcp"
      # Configure public network as StaticIP
      elsif network_type.downcase == "staticip"
        # puts "network private staticip: #{networkcfg}" # REMOVE: print
        if network_config.key?("ip")
          ip = network_config["ip"]
          vm.network "private_network", ip: ip
        else
          errormsg = "Campo vms[#{vmid}].network.private[#{privateid}].config.ip mandatorio"
          raise KeyError.new(errormsg)
        end
      # TODO: CONFIGURE IPv6
      # Configure private network manually
      elsif network_type.downcase == "manual"
        # puts "network private manual: #{networkcfg}" # REMOVE: print
        vm.network "private_network", auto_config: false
      else
        errormsg = "Campo vms[#{vmid}].network.private[#{privateid}].type:#{network_type} incorreto"
        raise KeyError.new(errormsg)
      end
    end
  end

  # Configure port forwarding
  if config.fetch("forward_ports", nil)
    fwportscfgs = config["forward_ports"]
    # puts "fwportscfgs: #{fwportscfgs}" # REMOVE: print
    if not fwportscfgs.is_a?(Array)
      fwportscfgs = [fwportscfgs]
    end
    fwportscfgs.each_with_index do |fwportscfg, fwportid|
      if not fwportscfg.fetch("guest", nil)
        errormsg = "Campo vms[#{vmid}].network.forward_ports[#{fwportid}].guest mandatorio"
        raise KeyError.new(errormsg)
      end
      if not fwportscfg.fetch("host", nil)
        errormsg = "Campo vms[#{vmid}].network.forward_ports[#{fwportid}].host mandatorio"
        raise KeyError.new(errormsg)
      end
      guest = fwportscfg["guest"]
      host = fwportscfg["host"]
      protocol = fwportscfg["protocol"] ? fwportscfg["protocol"]: "tcp"
      vm.network "forwarded_port", guest: guest, host: host, protocol: protocol
    end
  end
end

def provisioning(vm, vmid, config)
  # Configure Provision
  provisionings = convert_to_array(config)

  provisionings.each_with_index do |provisioning, provisionid|
    # puts "provisioning: #{provisioning}" # REMOVE: print
    # Validate type field
    if !provisioning.has_key?("type")
      errormsg = "Campo vms[#{vmid}].provisioning[#{provisionid}].type mandatorio"
      raise KeyError.new(errormsg)
    end

    # Configure name field
    if !provisioning.has_key?("name")
      provisioning["name"] = "Exec Provision Position: #{provisionid+1}"
    end

    # Configure run field
    if !provisioning.has_key?("run")
      provisioning["run"] = "once"
    end

    # Provisioning type = shell
    if provisioning["type"] == "shell"
      # Execute provision command shell
      if provisioning.has_key?("inline")
        vm.provision provisioning["name"],
          type: provisioning["type"],
          preserve_order: true,
          run: provisioning["run"],
          inline: provisioning["inline"]
      
      # Execute provision script shell
      elsif provisioning.has_key?("path")
        vm.provision provisioning["name"],
          type: provisioning["type"],
          preserve_order: true,
          run: provisioning["run"],
          inline: provisioning["path"]
      else
        raise "O Hash deve conter a chave 'inline' ou 'path'"
      end
    
    # Provisioning type = file
    elsif provisioning["type"] == "file"
      # Validate file fields
      if !provisioning.has_key?("source") && !provisioning.has_key?("destination")
        raise "O Hash deve conter a chave 'source' ou 'destination'"
      end
      vm.provision provisioning["name"],
        type: provisioning["type"],
        preserve_order: true,
        run: provisioning["run"],
        source: provisioning["source"],
        destination: provisioning["destination"]
    end
  end
end

# Build environment
def build(config, vms)
  # puts "VMS: #{vms}" # REMOVE: print
  # Loop in configurations of each VM
  vms.each_with_index do |vm, vmid|
    # puts "VM: #{vm}" # REMOVE: print
    name = vm["name"] ? vm["name"] : nil
    if not name
      raise KeyError.new("Campo 'name' mandatorio")
    end

    # Define VM Settings
    vm_cfg = default_settings(vm)

    # CentOS settings
    if vm["distro"] == "centos/7"
      config.vbguest.installer_options = { allow_kernel_upgrade: true }
    end
    
    # Configure VM
    config.vm.define name, primary: vm_cfg["primary"] do |server|
      server.vm.box = vm_cfg["distro"]
      server.vm.hostname = vm_cfg["hostname"]

      # Configure Synced Folders
      synced_folder_cfg = vm_cfg["synced_folder"] ? vm_cfg["synced_folder"] : {}
      synced_folder(server.vm, synced_folder_cfg)
      
      # Configure Disk
      if vm_cfg.fetch("disk", nil)
        disk_cfg = vm_cfg["disk"]
        disk(server.vm, disk_cfg)
      end
      
      # Configure Network
      if vm_cfg.fetch("network", nil)
        network_cfg = vm_cfg["network"]
        network(server.vm, vmid, network_cfg)
      end
      
      # Configure VirtualBox
      server.vm.provider "virtualbox" do |vb|
        vb.name = vm_cfg["name"]
        vb.memory = "%d" % [1024 * vm_cfg["ram"]]
        vb.cpus = vm_cfg["cpu"]
        vb.check_guest_additions = vm_cfg["guest_additions"]
      end

      # Provisioning
      if vm_cfg.fetch("provisioning", nil)
        provisioning_cfg = vm_cfg["provisioning"]
        provisioning(server.vm, vmid, provisioning_cfg)
      end
    end
  end
end


# Build Virtual Machine
Vagrant.configure("2") do |config|
  build(config, VMS)
end
