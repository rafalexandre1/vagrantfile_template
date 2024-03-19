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
  vms.each do |vm|
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
      server.vm.synced_folder ".", "/vagrant", disabled: vmcfg["sync_folder"]

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
          extra_disks.each_with_index do |diskcfg, index|
            disksize = diskcfg["size"] ? diskcfg["size"] : "20GB"
            diskname = diskcfg["name"] ? diskcfg["name"] : "extradisk-#{index+1}"
            # puts "disk cfg: #{disk_cfg}" # REMOVE: print
            # puts "diskname: #{diskname}" # REMOVE: print
            # puts "disksize: #{disksize}" # REMOVE: print
            server.vm.disk :disk, size: disksize, name: diskname
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
