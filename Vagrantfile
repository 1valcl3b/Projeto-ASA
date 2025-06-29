# Vagrantfile para o Projeto ASA - Ivalcleb e Pedro
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.ssh.insert_key = false
  config.vm.provider "virtualbox" do |vb|
    vb.linked_clone = true
    vb.check_guest_additions = false
  end

  # Servidor de Arquivos (arq)
  config.vm.define "arq" do |arq|
    arq.vm.hostname = "arq.ivalcleb.pedro.devops"
    arq.vm.network "private_network", ip: "192.168.56.113"

    arq.vm.provider "virtualbox" do |vb|
      vb.memory = 512
    end

    # Discos adicionais 
    (0..2).each do |x|
      arq.vm.disk :disk, size: "10GB", name: "disk-#{x}"
    end
  end

  # Servidor de Banco de Dados (db)
  config.vm.define "db" do |db|
    db.vm.hostname = "db.ivalcleb.pedro.devops"
    db.vm.network "private_network", type: "dhcp", mac: "aabbccddee01"
    db.vm.provider "virtualbox" do |vb|
      vb.memory = 512
    end
  end

  # Servidor de Aplicação (app)
  config.vm.define "app" do |app|
    app.vm.hostname = "app.ivalcleb.pedro.devops"
    app.vm.network "private_network", type: "dhcp", mac: "aabbccddee02"
    app.vm.provider "virtualbox" do |vb|
      vb.memory = 512
    end
  end

  # Cliente (cli)
  config.vm.define "cli" do |cli|
    cli.vm.hostname = "cli.ivalcleb.pedro.devops"
    cli.vm.network "private_network", type: "dhcp"
    cli.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
    end
  end
end

