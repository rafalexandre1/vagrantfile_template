# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

# CONFIGURAÇÃO PARA AS VMS
ENVIROMENT = YAML.load_file('enviroment.yml')
VMS = ENVIROMENT['vms']

# Converter configuração para lista de VMs caso necessario
if not VMS.is_a?(Array)
  VMS = [VMS]
end
# puts "VMS: #{VMS}" # REMOVE: print


def default_settings(vmdata)
  vm_config = {}
  default_config = {
    "hostname"          => vmdata["name"],
    "distro"            => "debian/buster64",
    "primary"           => false,
    "sync_folder"       => true,
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


# Contruir ambiente
def build(config, vms)
  # puts "VMS: #{vms}" # REMOVE: print
  ## Loop em configurações de cada VM
  vms.each_with_index do |vm, vmid|
    ## Validar variaveis definidas
    # puts "VM: #{vm}" # REMOVE: print
    name = vm["name"] ? vm["name"] : nil
    if not name
      raise KeyError.new("Campo 'name' mandatorio")
    end

    # Definir configurações da vm
    vmcfg = default_settings(vm)
    
    # Configurar a VM
    config.vm.define name, primary: vmcfg["primary"] do |server|
      server.vm.box = vmcfg["distro"]
      server.vm.hostname = vmcfg["hostname"]

      # Configurar Pastas Sincronizadas
      # Configurar pasta default
      syncdir_cfg = vmcfg["synced_folder"] ? vmcfg["synced_folder"] : {}
      syncdir_default = syncdir_cfg.key?("default_disabled") ? syncdir_cfg["default_disabled"] : false
      server.vm.synced_folder "./data", "/vagrant", disabled: syncdir_default
      
      # Configurar pastas extras
      if syncdir_cfg.fetch("extra", nil)
        sync_folders = syncdir_cfg["extra"]
        if not sync_folders.is_a?(Array)
          sync_folders = [sync_folders]
        end

        sync_folders.each_with_index do |sync_folder, folderid|
          type = sync_folder["type"] ? sync_folder["type"] : "rsync"
          target = sync_folder["target"]
          dest = sync_folder["dest"]

          # Configurar sync com type rsync
          disabled = sync_folder["disabled"] ? sync_folder["disabled"] : false
          if type == "rsync"
            server.vm.synced_folder target, dest, type: type, 
              disabled: disabled
          end
        end
      end

      # Configurar Disco
      if vmcfg.fetch("disk", nil)
        disk_cfg = vmcfg["disk"]
        # puts "disk: #{disk_cfg}" # REMOVE: print

        # Configurar disco primario
        if disk_cfg.fetch("primary", nil)
          diskcfg = disk_cfg["primary"]
          diskname = diskcfg["name"] ? diskcfg["name"] : "primary"
          disksize = diskcfg["size"] ? diskcfg["size"] : "20GB"
          # puts "disk cfg: #{disk_cfg}" # REMOVE: print
          # puts "diskname: #{diskname}" # REMOVE: print
          # puts "disksize: #{disksize}" # REMOVE: print
          server.vm.disk :disk, size: disksize, name: diskname, primary: true
        end

        # Configurar discos extras
        if disk_cfg.fetch("extra", nil)
          extra_disks = disk_cfg["extra"]
          if not extra_disks.is_a?(Array)
            extra_disks = [extra_disks]
          end
          extra_disks.each_with_index do |diskcfg, diskid|
            disksize = diskcfg["size"] ? diskcfg["size"] : "20GB"
            diskname = diskcfg["name"] ? diskcfg["name"] : "extradisk-#{diskid}"
            # puts "disk cfg: #{disk_cfg}" # REMOVE: print
            # puts "diskname: #{diskname}" # REMOVE: print
            # puts "disksize: #{disksize}" # REMOVE: print
            server.vm.disk :disk, size: disksize, name: diskname
          end
        end
      end

      # Configurar Network
      if vmcfg.fetch("network", nil)
        network_cfg = vmcfg["network"]
        # puts "network: #{network_cfg}" # REMOVE: print

        # Configurar public network
        if network_cfg.fetch("public", nil)
          networkcfg = network_cfg["public"]
          # puts "networkcfg: #{networkcfg}" # REMOVE: print
          network_type = networkcfg.fetch("type", nil)
          network_config = networkcfg["config"] ? networkcfg["config"] : {}

          # Configurar public network como DHCP
          if network_type.downcase == "dhcp"
            if network_config.key?("use_dhcp_assigned_default_route")
              dhcp_route = network_config.fetch("use_dhcp_assigned_default_route")
              # puts "network dhcp 1: #{network_cfg}" # REMOVE: print
              server.vm.network "public_network",
                use_dhcp_assigned_default_route: dhcp_route
            else
              # puts "network dhcp 2: #{network_cfg}" # REMOVE: print
              server.vm.network "public_network"
            end
          
          # Configurar public network como StaticIP
          elsif network_type.downcase == "staticip"
            if network_config.key?("ip")
              ip = network_config["ip"]
              server.vm.network "public_network", ip: ip
            else
              errormsg = "Campo vms[#{vmid}].network.public.config.ip mandatorio"
              raise KeyError.new(errormsg)
            end
          # Configurar public network como Bridge
          elsif network_type.downcase == "bridge"
            if network_config.fetch("interfaces", nil)
              interfaces = network_config["interfaces"]
              if not interfaces.is_a?(Array)
                interfaces = [interfaces]
              end
              server.vm.network "public_network", bridge: interfaces
            else
              errormsg = "Campo vms[#{vmid}].network.public.config.interfaces mandatorio"
              raise KeyError.new(errormsg)
            end
          # Configurar public network Manualmente
          elsif network_type.downcase == "manual"
            config.vm.network "public_network", auto_config: false
          else
            errormsg = "Campo vms[#{vmid}].network.public.type:#{network_type} incorreto"
            raise KeyError.new(errormsg)
          end
        end

        # Configurar private network
        if network_cfg.fetch("private", nil)
          networkcfgs = network_cfg["private"]
          # puts "networkcfgs: #{networkcfgs}" # REMOVE: print
          if not networkcfgs.is_a?(Array)
            networkcfgs = [networkcfgs]
          end
          networkcfgs.each_with_index do |networkcfg, privateid|
            network_type = networkcfg.fetch("type", nil)
            network_config = networkcfg["config"] ? networkcfg["config"] : {}
            
            # Configurar private network como DHCP
            if network_type.downcase == "dhcp"
              # puts "network private dhcp: #{networkcfg}" # REMOVE: print
              server.vm.network "private_network", type: "dhcp"
            # Configurar public network como StaticIP
            elsif network_type.downcase == "staticip"
              # puts "network private staticip: #{networkcfg}" # REMOVE: print
              if network_config.key?("ip")
                ip = network_config["ip"]
                server.vm.network "private_network", ip: ip
              else
                errormsg = "Campo vms[#{vmid}].network.private[#{privateid}].config.ip mandatorio"
                raise KeyError.new(errormsg)
              end
            # TODO: CONFIGURAR IPv6
            # Configurar private network Manualmente
            elsif network_type.downcase == "manual"
              # puts "network private manual: #{networkcfg}" # REMOVE: print
              config.vm.network "private_network", auto_config: false
            else
              errormsg = "Campo vms[#{vmid}].network.private[#{privateid}].type:#{network_type} incorreto"
              raise KeyError.new(errormsg)
            end
          end
        end

        # Configurar redirecionamento de porta
        if network_cfg.fetch("forward_ports", nil)
          fwportscfgs = network_cfg["forward_ports"]
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
            server.vm.network "forwarded_port", guest: guest, host: host, protocol: protocol
          end
        end
      end

      # Configurar VirtualBox
      server.vm.provider "virtualbox" do |vb|
        vb.name = vmcfg["name"]
        vb.memory = "%d" % [1024 * vmcfg["ram"]]
        vb.cpus = vmcfg["cpu"]
        vb.check_guest_additions = vmcfg["guest_additions"]
      end

    end
  end
end


# All Vagrant configuration
Vagrant.configure("2") do |config|
  build(config, VMS)
end
