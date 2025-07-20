# Vagrantfile para o Projeto ASA - Ivalcleb e Pedro
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.ssh.insert_key = false

  # Verifica e desativa o auto update do vbguest
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end

  # Desabilita pasta /vagrant compartilhada
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Configuração do provedor padrão
  config.vm.provider :virtualbox do |vb|
    vb.memory = 512
    vb.linked_clone = true
    vb.check_guest_additions = false
  end

  # VM: Servidor de Arquivos (arq)
  config.vm.define "arq" do |arq|
    arq.vm.hostname = "arq.ivalcleb.pedro.devops"
    arq.vm.network :private_network, ip: "192.168.56.113"

    # Discos adicionais
    (0..2).each do |x|
      arq.vm.disk :disk, size: "10GB", name: "disk-#{x}"
    end

    arq.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/dhcp-arq.yml"
      ansible.extra_vars = { target: "arq" }
    end

    arq.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/lvm-arq.yml"
      ansible.extra_vars = { target: "arq" }
    end

    arq.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/nfs-arq.yml"
      ansible.extra_vars = { target: "arq" }
    end
  end

  # VM: Servidor de Banco de Dados (db)
  config.vm.define "db" do |db|
    db.vm.hostname = "db.ivalcleb.pedro.devops"
    db.vm.network :private_network, type: "dhcp", mac: "08:00:27:3a:50:5b"

    db.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/db.yml"
      ansible.extra_vars = { target: "db" }
    end
  end

  # VM: Servidor de Aplicações (app)
  config.vm.define "app" do |app|
    app.vm.hostname = "app.ivalcleb.pedro.devops"
    app.vm.network :private_network, type: "dhcp", mac: "08:00:27:3a:50:5c"

    app.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/app.yml"
      ansible.extra_vars = { target: "app" }
    end
  end

  # VM: Cliente (cli)
  config.vm.define "cli" do |cli|
    cli.vm.hostname = "cli.ivalcleb.pedro.devops"
    cli.vm.network :private_network, type: "dhcp"
    cli.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
    end

    cli.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.playbook = "playbooks/cli.yml"
      ansible.extra_vars = { target: "cli" }
    end
  end

  # Playbook base comum a todas as VMs
  config.vm.provision "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "playbooks/base.yml"
  end
end
