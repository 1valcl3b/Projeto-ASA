  Projeto DevOps com Vagrant e Ansible
Integrantes da equipe: Pedro Henrique Remigio Fernandes Thomaz e Ivalcleb
Disciplina: Administração de Sistemas Abertos
Professor: Leonidas Francisco de Lima Júnior

  Introdução
Este projeto tem como objetivo o provisionamento e configuração automática de uma infraestrutura composta por quatro máquinas virtuais Linux (Debian), utilizando as ferramentas Vagrant e Ansible. O ambiente é configurado para simular um cenário real de DevOps, envolvendo servidores de arquivos, banco de dados, aplicação e um host cliente, com automação de serviços como SSH, NFS, LVM, DHCP, Apache e MariaDB. Toda a documentação, bem como os arquivos de configuração (Vagrantfile e playbooks), está descrita e incluída neste repositório.

  Vagrantfile
O projeto inclui um Vagrantfile que define a criação de quatro máquinas virtuais:
Servidor de Arquivos (arq): hostname arq.ivalcleb.pedro.devops, IP estático 192.168.56.113, 512 MB de RAM, três discos adicionais de 10 GB cada.
Servidor de Banco de Dados (db): hostname db.ivalcleb.pedro.devops, IP via DHCP, 512 MB de RAM.
Servidor de Aplicação (app): hostname app.ivalcleb.pedro.devops, IP via DHCP, 512 MB de RAM.
Cliente (cli): hostname cli.ivalcleb.pedro.devops, IP via DHCP, 1024 MB de RAM.
Todas as máquinas usam a box debian/bookworm64, com linked_clone habilitado, guest additions desabilitado e sem geração de novas chaves SSH.

  Playbook app.yml
O playbook app.yml é responsável pela configuração do servidor de aplicação. Ele executa as seguintes tarefas:
Instala o servidor web Apache2.
Substitui a página padrão do Apache por um index.html customizado com a descrição do projeto e dados dos integrantes.
Instala o serviço autofs.
Configura o autofs para montagem automática do compartilhamento NFS exportado pelo servidor de arquivos no diretório /var/nfs.
Reinicia e habilita o serviço autofs para garantir a montagem automática.
A página HTML contém informações da disciplina, professor e dos integrantes, conforme solicitado no projeto.

  Playbook de configuração base (base.yml)
O playbook base.yml realiza a configuração comum a todas as VMs. Ele inclui:
Atualização do sistema (update e upgrade).
Instalação e configuração do chrony apontando para pool.ntp.br.
Configuração do timezone para America/Recife.
Criação do grupo ifpb.
Criação dos usuários ivalcleb e pedro, configurando seus diretórios .ssh e gerando chaves SSH.
Configuração do SSH para permitir apenas autenticação por chaves públicas, bloqueio do root, banner de aviso e limitação de acesso aos grupos vagrant e ifpb.
Instalação do cliente NFS.
Configuração do sudo para o grupo ifpb.

  Playbook cli.yml
O playbook cli.yml realiza a configuração do host cliente. Ele executa as seguintes tarefas:
Instala o firefox-esr e xauth para suporte a aplicações gráficas.
Ativa o encaminhamento X11 no SSH para permitir exportação da interface gráfica de aplicativos.
Reinicia o serviço SSH para aplicar as alterações.
Instala e configura o autofs para montagem automática do compartilhamento NFS exportado pelo servidor de arquivos no diretório /var/nfs.
Cria o ponto de montagem local e reinicia o serviço autofs para garantir o funcionamento da montagem automática.
  
  Playbook db.yml
O playbook db.yml realiza a configuração do servidor de banco de dados. Ele executa as seguintes tarefas:
Instala o servidor de banco de dados mariadb-server.
Instala o autofs.
Configura o autofs para montagem automática do compartilhamento NFS exportado pelo servidor de arquivos no diretório /var/nfs.
Cria o ponto de montagem local e reinicia o serviço autofs para garantir a montagem automática do diretório remoto.
  
  Playbook arq.yml
O playbook arq.yml realiza a configuração do servidor de arquivos. Ele executa as seguintes tarefas:
Instala o pacote isc-dhcp-server.
Configura a interface do serviço DHCP.
Cria o arquivo de configuração dhcpd.conf definindo o servidor como autoritativo, configurando o domínio, servidores DNS, range de endereços para DHCP e reservas de IPs para os hosts db e app baseados em seus endereços MAC.
Reinicia e habilita o serviço DHCP.
  
  Playbook lvm_arq.yml
O playbook lvm_arq.yml realiza a configuração do LVM no servidor de arquivos. Ele executa as seguintes tarefas:
Instala os pacotes lvm2 e parted.
Cria partições nos discos adicionais (sdb, sdc, sdd).
Inicializa volumes físicos em cada partição.
Cria o volume group dados utilizando os três discos.
Cria o logical volume ifpb com 15 GB de tamanho.
Formata o logical volume com o sistema de arquivos ext4.
Cria o ponto de montagem /dados.
Adiciona entrada no /etc/fstab para montagem automática do volume logical no diretório /dados.

  Playbook nfs_arq.yml
O playbook nfs_arq.yml configura o servidor NFS no servidor de arquivos. Ele executa as seguintes tarefas:
Instala o servidor NFS (nfs-kernel-server).
Cria o usuário nfs-ifpb sem shell para maior segurança.
Cria o diretório compartilhado /dados/nfs, definindo o usuário e grupo como nfs-ifpb.
Obtém o UID e GID do usuário nfs-ifpb e armazena em variáveis.
Configura o arquivo /etc/exports para exportar o diretório compartilhado para a rede 192.168.56.0/24, com permissões de leitura e escrita, squash de todos os usuários remotos para nfs-ifpb, e sincronização imediata das gravações em disco.
Reinicia e habilita o serviço NFS.

  Playbook ssh_config.txt
O playbook ssh_hosts_config configura as conexões SSH para quatro máquinas virtuais locais nomeadas arq, db, app e cli. Ele executa as seguintes tarefas:
Define o host arq com IP 127.0.0.1 e porta 2222, configurando o usuário vagrant e o uso de chaves privadas para autenticação, desabilitando a verificação estrita de chaves e a autenticação por senha.
Define o host db com IP 127.0.0.1 e porta 2200, utilizando as mesmas configurações de usuário, autenticação por chave e opções de segurança que o host anterior.
Configura o host app com IP 127.0.0.1 e porta 2201, mantendo a mesma estrutura de autenticação e segurança, permitindo conexão automática sem intervenção manual.
Configura o host cli com IP 127.0.0.1 e porta 2202, seguindo os mesmos padrões de autenticação por chave, desativação da verificação de host e log reduzido para facilitar conexões rápidas e seguras.
Em todas as configurações, são especificados os arquivos de chave privada padrão do Vagrant, garantindo autenticação segura sem uso de senha, além de aceitar chaves do tipo ssh-rsa para compatibilidade.
Essa configuração simplifica o acesso SSH às máquinas virtuais locais, facilitando operações automatizadas e o gerenciamento da infraestrutura virtual.


