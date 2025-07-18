# Projeto DevOps com Vagrant e Ansible

**Integrantes da equipe:** Ivalcleb Leoncio Benigno de Souza(20232380013) e Pedro Henrique Remigio Fernandes Thomaz(20232380007)

**Disciplina:** Administração de Sistemas Abertos

**Professor:** Leonidas Francisco de Lima Júnior

## Introdução
Este projeto tem como objetivo o provisionamento e configuração automática de uma infraestrutura composta por quatro máquinas virtuais Linux (Debian), utilizando as ferramentas Vagrant e Ansible. O ambiente é configurado para simular um cenário real de DevOps, envolvendo servidores de arquivos, banco de dados, aplicação e um host cliente, com automação de serviços como SSH, NFS, LVM, DHCP, Apache e MariaDB. Toda a documentação, bem como os arquivos de configuração (Vagrantfile e playbooks), está descrita e incluída neste repositório.

## Vagrantfile

O projeto inclui um Vagrantfile que define a criação de quatro máquinas virtuais:

- Servidor de Arquivos (arq): hostname arq.ivalcleb.pedro.devops, IP estático 192.168.56.113, 512 MB de RAM, três discos adicionais de 10 GB cada.

- Servidor de Banco de Dados (db): hostname db.ivalcleb.pedro.devops, IP via DHCP, 512 MB de RAM.

- Servidor de Aplicação (app): hostname app.ivalcleb.pedro.devops, IP via DHCP, 512 MB de RAM.

- Cliente (cli): hostname cli.ivalcleb.pedro.devops, IP via DHCP, 1024 MB de RAM.
  
Todas as máquinas usam a box debian/bookworm64, com linked_clone habilitado, guest additions desabilitado e sem geração de novas chaves SSH.

## Playbooks 

Os playbooks foram organizados de acordo com as responsabilidades de cada máquina:

- **`base.yml`**  
  Aplica configuração comum a todas as VMs:  
  - Atualização do sistema  
  - Instalação e configuração do Chrony (NTP)  
  - Ajuste de timezone para `America/Recife`  
  - Criação de grupo `ifpb` e usuários `ivalcleb` e `pedro`  
  - Configuração de chaves SSH, sudo e segurança do SSH (bloqueio root, senha desativada, banner)  
  - Instalação do cliente NFS  

- **`dhcp-arq.yml`**  
  Configura o servidor DHCP na VM `arq`:  
  - Instalação do pacote `isc-dhcp-server`  
  - Configuração do arquivo `/etc/dhcp/dhcpd.conf` com faixa `192.168.56.50-192.168.56.100`  
  - Reservas para VMs `db` e `app` por MAC address  
  - Reinicialização do serviço DHCP  

- **`lvm-arq.yml`**  
  Configura LVM na VM `arq`:  
  - Criação de volume group (`dados`) e logical volume (`ifpb` com 15GB)  
  - Montagem em `/dados` e atualização do `/etc/fstab`  

- **`nfs-arq.yml`**  
  Configura NFS na VM `arq`:  
  - Instalação do `nfs-kernel-server`  
  - Criação do diretório `/dados/nfs` com permissões  
  - Configuração do `/etc/exports` para exportar a pasta via NFS  

- **`db.yml`**  
  Configura a VM `db`:  
  - Instalação do MariaDB  
  - Configuração do `autofs` para montar `/var/nfs` automaticamente via NFS  

- **`app.yml`**  
  Configura a VM `app`:  
  - Instalação do Apache  
  - Substituição da página padrão por `index.html` personalizado  
  - Configuração do `autofs` para montagem via NFS  

- **`cli.yml`**  
  Configura a VM `cli`:  
  - Instalação do Firefox ESR e `xauth`  
  - Ativação do X11 Forwarding no SSH  
  - Configuração do `autofs` para montagem via NFS  


## Execução do Projeto

### **Pré-requisitos**
- **VirtualBox** instalado  
- **Vagrant** instalado  
- **Ansible** instalado no host  
  

### **Passos para execução**

1. **Clone este repositório:**
   ```bash
   git clone https://github.com/1valcl3b/Projeto-ASA
   cd Projeto-ASA
   
2. Desative o DHCP padrão do VirtualBox (importante):
   ```bash
   vboxmanage dhcpserver stop --interface=vboxnet0

3. Suba as VMs na ordem correta:
   ```bash
   vagrant up arq
   vagrant up db app cli
