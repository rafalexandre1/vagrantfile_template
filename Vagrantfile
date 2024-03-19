# -*- mode: ruby -*-
# vi: set ft=ruby :

# CONFIGURAÇÃO PARA AS VMS
VMS = [
  {
    hostname: "PLAYGROUND",                     # obrigatorio
    distro: "debian/buster64",                  # opcional - default: "debian/buster64"
    primary: false,                             # opcional - default: false
    sync_folder: false,                         # opcional - default: true
    disk: "60GB",                               # opcional - default: "20GB"
    ram: 2,                                     # opcional - default: 2
    cpu: 2,                                     # opcional - default: 2
    guest_additions: true                       # opcional - default: true
  }
]

# Construtor de cluster
  # def cluster(config, hostname, forwarded_ports, distro = "debian/buster64", ram = 2, disk = 50, inw = "eno1")
def build(config, vms)
  ## Loop em configurações de cada VM
  vms.each do |vm|
    ## Validar variaveis definidas
    # TODO: Criar a logica para validar

    # Definir as variaveis
    hostname = vm[:hostname]
    distro = vm[:distro] ? vm[:distro] : "debian/buster64"
    primary = vm[:primary] ? vm[:primary] : false
    sync_folder = vm[:sync_folder] ? vm[:sync_folder] : true
    disk = vm[:disk] ? vm[:disk] : "20GB"
    ram = vm[:ram] ? vm[:ram] : 2
    cpu = vm[:cpu] ? vm[:cpu] : 2
    guest_additions = vm[:guest_additions] ? vm[:guest_additions] : true

    # Configurar a VM
    config.vm.define hostname, primary: primary do |server|
      server.vm.box = distro
      server.vm.hostname = hostname
      server.vm.synced_folder ".", "/vagrant", disabled: sync_folder
      server.vm.disk :disk, size: disk, primary: true

      # Configurar VirtualBox
      server.vm.provider "virtualbox" do |vb|
        vb.name = hostname
        vb.memory = "%d" % [1024 * ram]
        vb.cpus = cpu
        vb.check_guest_additions = guest_additions
      end

    end
  end    
end


# All Vagrant configuration
Vagrant.configure("2") do |config|
  build(config, VMS)
end
