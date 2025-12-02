<p align="center">
<img 
    src=".github/assets/images/readme/cover/cover.jpg"
    width="300"
/>
</p>

<p align="center">
  <a href="https://ffinfiniti.com/">
    <img src="https://img.shields.io/badge/Apache-HTTP%20Server-D22128?style=for-the-badge&logo=apache&logoColor=white" alt="Apache">
    <img src="https://img.shields.io/badge/PHP-Backend-777BB4?style=for-the-badge&logo=php&logoColor=white" alt="PHP">
    <img src="https://img.shields.io/badge/GLPI-ITSM-F9A825?style=for-the-badge&logo=glpi&logoColor=white" alt="GLPI">
    <img src="https://img.shields.io/badge/MkDocs-Docs-000000?style=for-the-badge&logo=mkdocs&logoColor=white" alt="MkDocs">
    <img src="https://img.shields.io/badge/MySQL-Database-4479A1?style=for-the-badge&logo=mysql&logoColor=white" alt="MySQL">
    <img src="https://img.shields.io/badge/GRAFANA-ZABBIX%20Server-D22128?style=for-the-badge&logo=grafana&logoColor=white" alt="Zabbix">
  </a>

# Install Apache, PHP, Lets Encrypt, GLPI, MkDocs, Zabbix e Grafana com o Ansible 

Projeto para instalação de servidor apache, php, letsencrypt, GLPI, MkDocs, Zabbix integrdo com o Grafana utilizando o Ansible.

## Links de referência

### Ansible
- [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#upgrading-ansible)

### RHEL Ansible
- [Yum module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html#ansible-collections-ansible-builtin-yum-module)
- [Dnf module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/dnf_module.html#requirements)
- [Systemd module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html#examples)
- [Firewalld module](https://docs.ansible.com/ansible/latest/collections/ansible/posix/firewalld_module.html)
- [Lets Encrypt](https://docs.ansible.com/ansible/2.5/modules/letsencrypt_module.html)
- [Ansigle Galaxy Crypto](https://galaxy.ansible.com/community/crypto)
- [community.crypto.acme](https://docs.ansible.com/ansible/latest/collections/community/crypto/acme_certificate_module.html)
- [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-acquire-a-let-s-encrypt-certificate-using-ansible-on-ubuntu-18-04)
- [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-20-04)

## Fases do Projeto
```
- Provisioning => Criar as instâncias/vms para o nosso server.
- Install => Instalação do Apache, PHP, GLPI, LetsEncrypt, MkDocs, Zabbix e Grafana.
- Deploy_apps => Deploy de aplicações
```

## Pre Requisitos

- Para o Lets Encrypt, caso de algum erro leia a documentação e realize os ajustes manualmente:
    * Tem que ter todos os apontamentos no dns (Seu servidor tem que estar disponivel na web nas portas 80 e 443);
    * Comando para gerar todos os certificados:
        $ sudo certbot --nginx
        $ sudo certbot --apache
        $ sudo systemctl restart httpd

- Utilização do Ansible
  - Em meu cenário tenho o WSL em meu Sistema Operacional e uma Distro do Ubuntu 
  - Leia a documentação de como instalar [Documentação](https://learn.microsoft.com/pt-br/windows/wsl/install)
  - Caso utilize alguma distro linux basta seguir a instalação do ansible [Documentação](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu).
    
    ```
    $ sudo apt update
    $ sudo apt install software-properties-common
    $ sudo add-apt-repository --yes --update ppa:ansible/ansible
    $ sudo apt install ansible
    $ sudo apt install ansible-core
    $ ansible-galaxy collection list
    $ ansible-galaxy collection install ansible.posix
    $ ansible-galaxy collection install t_systems_mms.letsencrypt #deprecated
    $ ansible-galaxy collection install community.crypto
    $ ansible-galaxy collection install community.mysql
    $ ansible-galaxy collection install community.general
    
    ```

## Clonando o repositório e executando o Ansible Passo a Passo

- Clone esse repositório para seu PC.

### Criar uma VM linux

- Crie uma máquina virtual linux (Sugestão utilize a distro Almalinux minimal);
- Ou se estiver no ambiente windows e quiser criar outra instancia com WSL (Sugiro tambem utilizar a distro Almalinux)
- Definir o hostname da distro para vm-node-02, se necessário insira em seu hosts da distro com o ansible para que ele consiga se comunicar com a vm através do nome vm-node-02.
  - Criar arquivo hosts para testar a comunicação entre os servers (Esse arquivo deve ser criado em sua maquina que fará a comunicação com o host)
    
    ```
    $ sudo vi /etc/hosts
    Add os ips ou nome dos hosts no arquivo.
    ```

### Caso optou pelo WSL parar a distro Almalinux

- Alguns pontos
  - A versão do windows tem que ser Windows 11 23H2 ou superior;
  - WSL version: 2.1.x Kernel version: 5.15.x ou superior;
  - Definir todas as distros para a versão 2
    ```
    wsl -l -v
    wsl --set-version <NomeDaDistro> 2 (Realize para todas as distros)
    ```

- Em seu windows crie o arquivo .wslconfig para que cada distro tenha um ip independente
    ```
    #powershell
    notepad $env:USERPROFILE\.wslconfig

    #Adicione no arquivo o conteudo abaixo
    [wsl2]
    networkingMode=mirrored
    ```

- Depois reinicie o WSL completamente:
    ```
    wsl --shutdown
    ```
- Abra suas distros novamente, confira se a eth0 está com ip e ativa
    ```
    ip addr show eth0
    ```

- Se a interface estiver down
    ```
    sudo ip link set eth0 up
    sudo dhclient eth0
    ```

- Alterar hostname da distro
  - Execute na distro o comando
    ```
    sudo bash -c 'echo -e "[network]\nhostname = vm-node-02\ngenerateResolvConf = false" > /etc/wsl.conf'
    sudo hostnamectl set-hostname vm-node-02
    ```

### Copiando a chave RSA

- Criar uma chave RSA
  - Em seu ubuntu com o ansible instalado gere a chave RSA e copie para a distro que será nosso servidor
  - Ao copiar a chave administrator@vm-node-02, onde administrator se refere ao nome do usuario da distro
  - Se alterar o nome do usuário terá que alterar também nos arquivos .yml.
    
    ```
    $ ssh-keygen    
    $ ssh-copy-id -i ~/.ssh/id_rsa administrator@vm-node-02
    ```
    - Após copiar teste o acesso ssh
        ```
        ssh 'administrator@vm-node-02'
        ```

### Configurar acesso sem senha

- Remover solicitação de senha nos hosts
  - Realizar apenas nos hosts que serão executados o ansible (exemplo vm-node-02)
    
    ```
    $ ssh 'administrator@vm-node-02'
    $ sudo vi /etc/sudoers
    $ #User privilege specification
    $ administrator  ALL=(ALL:ALL) NOPASSWD:ALL
    $ # Verifique os grupos do usuario tambem
    $ whell  ALL=(ALL:ALL) NOPASSWD:ALL
    ```
  - Reinicie o servidor para aplicar a alteração do sudoers

### Realizando teste

- No servidor que instalamos o ansible rode o comando ad hoc de teste ansible abaixo
- Acesse a pasta "seu_caminho_para_a_pasta\rhel-apache-glpi-mkdocs-php-zabbix\provisioning" 
    
    ```
    $ ansible -i hosts all -m ping
    A saída desse comando tem que ser positiva.
    ```

### Executando todas as roles

- `Informações importantes antes de executar o comando:`
  - Provisioning
    - Realiza o update e instalação de pacotes necessários.
  - Install Apache
    - Considere alterar seus dados na role install_apache > files
    - Adicione as informações de seu ambiente:
      - o arquivo auth.conf é utilizado para solicitar autenticação em paginas que deseja adicionar restrições;
      - o arquivo authorized_passwd é a senha utilizada pelo usuario em auth.conf para vincular seu AD com o Apache.
    - O arquivo security.conf tem algumas configurações de segurança padrão
  - Install MkDocs
    - Considere alterar o arquivo em install_mkdocs > files > manualinfra.conf
        - Adicione seus dados de dominio;
    - Considere alterar o arquivo em install_mkdocs > vars > main.yml
        - Altere para seus dados de dominio entre outros que achar necessário alterar;
    - Tem uma pagina de exemplo criada (Após execuar com exito acesse o servidor http://IP-DO-SERVIDOR/manualinfra/)
    - Caso tenha interesse extraia e edite o arquivo em install_mkdocks > files > manualinfra.zip
      - Está bem estruturado e você pode adicionar as informações de sua infra essa é uma ótima ferramenta para documentação.
  - Install MySQL
    - Considere alterar seus dados na role install_mysql > files
      - Adicione as informações de seu ambiente:
      - o arquivo my.cnf contém o usuário e a senha cadastrada para o mysql.
      - Altere também a senha "P@sswd" em  install_mysql > tasks > install.yml (linhas 60, 68 e 72)
        - As senhas são para o usuário root e infra que é criado na execução dessa role
      - Aqui também já é criado o database do glpi
      - Caso precise definir a senha root do mysql manual
          ```
          $ mariadb-secure-installation
          ```
  - Install Apps
    - Atenção:
      - Caso não tenha um servidor DNS configurado para acessar as aplicações adicione em seu computador local.
        - Windows: c:\Windows\System32\drivers\etc\hosts
          - 192.168.135.139 esse é o ip do meu servidor vm-node-02 no momento me que gerei a VM de teste, altere para seu ip
          - 192.168.135.139 intranet.yourdomain.com.br
          - 192.168.135.139 manualinfra.yourdomain.com.br
          - 192.168.135.139 glpi.yourdomain.com.br
          - Caso tenha mais aplicações faça o mesmo
        - Linux: /etc/hosts
          - 192.168.135.139 esse é o ip do meu servidor vm-node-02 no momento me que gerei a VM de teste, altere para seu ip
          - 192.168.135.139 intranet.yourdomain.com.br
          - 192.168.135.139 manualinfra.yourdomain.com.br
          - 192.168.135.139 glpi.yourdomain.com.br
          - Caso tenha mais aplicações faça o mesmo

    - Intranet
      - Considere alterar o arquivo em install_intranet > files > intranet.conf
        - Adicione seus dados de dominio;
        - Para habilitar a segurança por grupo do ad descomente o bloco <Location> (Deve estar bem configurado sua restrição pelo AD para funcionar)
      - Considere alterar o arquivo em install_intranet > vars > main.yml
        - Altere para seus dados de dominio entre outros que achar necessário alterar;
    - GLPI
      - Caso vá utilizar o inventory do glpi para equipamentos sw, nas, e esxi descomente os arquivos .sh  em > install_glpiti > templates.
        - Basta descomentar as linhas desejadas e configurar com seu usuario e senha e commutity para os equipamentos que utiliza SNMP. 
      - Após a instalação basta abrir a pagina e seguir a configuração (glpi.yourdomain.com.br):
        - Usuário e senha do banco glpi caso não alterou na role install_mysql:
          - user: infra
          - passwd: P@sswd
      - Passo a passo da instalação após concluir a instalação de todas as roles está no final desse README [GLPI Click aqui](#glpi-instalação).

  - Install Zabbix
    - Atenção:
      - Caso não tenha um servidor DNS configurado para acessar as aplicações adicione em seu computador local.
        - Windows: c:\Windows\System32\drivers\etc\hosts
          - 192.168.135.139 esse é o ip do meu servidor vm-node-02 no momento me que gerei a VM de teste, altere para seu ip
          - 192.168.135.139 zabbix.yourdomain.com.br
          - Caso tenha mais aplicações faça o mesmo
        - Linux: /etc/hosts
          - 192.168.135.139 esse é o ip do meu servidor vm-node-02 no momento me que gerei a VM de teste, altere para seu ip
          - 192.168.135.139 zabbix.yourdomain.com.br
          - Caso tenha mais aplicações faça o mesmo

    - Zabbix
      - Considere ajustar os valores dos arquivos em install_zabbix > files
        - Altere para seus dados de dominio, senhas entre outros que achar necessário alterar;
      - Considere ajustar os valores do arquivo em install_zabbix > vars > main.yml
        - Altere para seus dados de dominio entre outros que achar necessário alterar;
      - Após a instalação basta abrir a pagina e seguir a configuração (zabbix.yourdomain.com.br):
      - Passo a passo da instalação após concluir a instalação de todas as roles está no final desse README [ZABBIX Click Aqui](#zabbix-instalação).
  - Install grafana
    - Atenção:
      - Caso não tenha um servidor DNS configurado para acessar as aplicações adicione em seu computador local.
        - Windows: c:\Windows\System32\drivers\etc\hosts
          - 192.168.135.139 esse é o ip do meu servidor vm-node-02 no momento me que gerei a VM de teste, altere para seu ip
          - 192.168.135.139 grafana.yourdomain.com.br
          - Caso tenha mais aplicações faça o mesmo
        - Linux: /etc/hosts
          - 192.168.135.139 esse é o ip do meu servidor vm-node-02 no momento me que gerei a VM de teste, altere para seu ip
          - 192.168.135.139 grafana.yourdomain.com.br
          - Caso tenha mais aplicações faça o mesmo
      
    - Considere ajustar os valores dos arquivos em install_grafana > files
        - Altere para seus dados de dominio, senhas entre outros que achar necessário alterar;
      - Considere ajustar os valores do arquivo em install_grafana > vars > main.yml
        - Altere para seus dados de dominio entre outros que achar necessário alterar, incluindo o acesso ao zabbix que será utilizado na integração com o grafana;
    
    - Após instalação acesse a URL grafana.yourdomain.com.br no fim desse manual tem um passo a passo [GRAFANA Click aqui](#grafana-instalação).

- Execute o comando abaixo já está configurado para executar todas as roles
 
    ```
    $ cd rhel-apache-glpi-mkdocs-php-zabbix
    $ ansible-playbook -i hosts main.yml
    ```
  
  - Após a execução acima se não ocorreu nenhum erro todas as roles foram instaladas com sucesso!

## Caso tenha curiosidade de saber o processo de criação das pastas e roles do Ansible passo a passo

- Nesse processo você vai aprender a gerar as pastas com comandos do ansible nesse processo já é gerado as pastas e arquivos .yml de cada role no padrão de projeto ansible.
- Depois de gerado basta criar seus playbooks.
- Gosto de fazer dessa forma pois já inicia um projeto bem estruturado e organizado. 

### Criar Roles

- Nas distribuições RHEL Foi necessário instalar o python-firewall para manipular o firewalld com o ansible.

- Nas distribuições RHEL Foi necessário instalar o libselinux-python para manipular o linux com o ansible.

- Nas distribuições RHEL Foi necessário instalar o python-dnf para manipular package manager dnf.

- Nas distribuições RHEL Foi necessário instalar o pexpect para manipular package expect.

- Criar pastas e arquivos necessários 'rhel-apache-glpi-mkdocs-php-zabbix'

    ```
    $ mkdir rhel-apache-glpi-mkdocs-php-zabbix
    $ cd rhel-apache-glpi-mkdocs-php-zabbix
    $ mkdir roles
    $ touch hosts
    $ touch main.yml
    ```

- Criar roles

    ```
    $ cd rhel-apache-glpi-mkdocs-php-zabbix\roles
    $ ansible-galaxy init provisioning
    $ ansible-galaxy init install_apache
    $ ansible-galaxy init install_php
    $ ansible-galaxy init install_letsencrypt
    $ ansible-galaxy init install_mkdocs
    $ ansible-galaxy init install_mysql
    $ ansible-galaxy init install_intranet
    $ ansible-galaxy init install_glpiti
    $ ansible-galaxy init install_zabbix
    $ ansible-galaxy init install_grafana
    ```

- Adicionar configuração arquivo hosts

    ```
    $ vi rhel-apache-glpi-mkdocs-php-zabbix\hosts
    $ Add
    [master]
    vm-node-02
    
    [master:vars]
    # vars domain
    domain="yourdomain.com.br"
    
    # vars php
    php_version_1=php74
    php_version_2=php83
    
    # vars file for config-sites
    path_apache_conf="/etc/httpd/conf.d"
    path_html="/var/www/html/"
    path_mkdocs="/var/www/mkdocs/"
    
    # vars git
    # github_user="youruser"
    # github_passwd="yourkey"
    ```

### Install Apache

- Considere alterar seus dados na role install_apache > files
  - Adicione as informações de seu ambiente:
    - o arquivo auth.conf é utilizado para solicitar autenticação em páginas que deseja adicionar restrições;
    - o arquivo authorized_passwd é a senha utilizada pelo usuário em auth.conf para vincular seu AD com o Apache.

- O arquivo security.conf tem algumas configurações de segurança padrão 

- Alguns sites para verificar a segurança de seu servidor apache:

    - Site scan dnssec
      - [dnssec-analyzer](https://dnssec-analyzer.verisignlabs.com/)
      - [configuracao_dnssec](https://ftp.registro.br/pub/doc/configuracao_dnssec_dominio.pdf)

    - Site scan headers
      - [securityheaders](https://securityheaders.com)

    - Site scan ssl
      - [ssllabs](https://www.ssllabs.com/ssltest/analyze.html)

    - Site Check cadeia de certificado
      - [whatsmychaincert](https://whatsmychaincert.com/)
      - [sslshopper](https://www.sslshopper.com/ssl-checker.html)

### Install Let's Encrypt

- Considere alterar seus dados na role install_letsencrypt > vars
  - Adicione as informações de seu ambiente:

- Observação 
    ```
    Para o LetsEncrypt criei em meu dns o apontamento para app.mydomain.com e www.app.mydomain.com;
    Em minha configuração do Apache não criei nenhum virtualhost para que ao executar o letsencrypt,
    fosse possível realizar a verificação da pagina http://app.mydomain.com/.well-known/acme-challenge/.

    ```

- Para solicitar certificados para seus sites basta exetutar em seu servidor

    ```
    $ sudo certbot --apache
    ```

- Gerar Certificado com o arquiv pfx
  ~~~shell
  openssl pkcs12 -in /home/usuario/meu_certificado.pfx -clcerts -nokeys -out certificado.crt
  openssl pkcs12 -in /home/usuario/meu_certificado.pfx -nocerts -nodes -out chave.key
  openssl pkcs12 -in /home/usuario/meu_certificado.pfx -cacerts -nokeys -out cadeia_intermediaria.crt
  ~~~

### Install MKdocs

- Links úteis
    * https://pypi.org/project/mkdocs/
    * https://github.com/mkdocs/catalog
    * https://github.com/zhaoterryy/mkdocs-pdf-export-plugin
    * https://doc.courtbouillon.org/weasyprint/latest/first_steps.html#ubuntu-20-04
    * https://github.com/orzih/mkdocs-with-pdf

### Install MYSQL

- Instalar python-PyMySQL para executar os comandos de mysql pelo ansible
     ```
    $ dnf -y python-PyMySQL
     ```

- Definir a senha root manual
    $ mariadb-secure-installation

### Install Apps

- Para realizar o acesso externo, deve se criar um apontamento no DNS de hospedagem para o nosso servidor interno.

- Deve realizar também o apontamento de DNS em nosso servidor de dominio interno, utilize o mesmo nome cadastrado de acesso externo.

#### Install Intranet
- Considere alterar o arquivo em install_intranet > files > intranet.conf
  - Adicione seus dados de dominio;
  - Para habilitar a segurança por grupo do ad descomente o bloco <Location> (Deve estar bem configurado sua restrição pelo AD para funcionar)

#### Install GLPI

- Comandos caso precise realizar Update das tables GLPI
    ```
    $ cd /var/www/html/glpi
    $ sudo php83 bin/console glpi:migration:utf8mb4
    $ sudo php83 bin/console glpi:migration:unsigned_keys

    $ sudo php83 bin/console database:check_schema_integrity
    $ sudo php83 bin/console migration:timestamps
    $ sudo php83 bin/console migration:utf8mb4
    $ sudo php83 bin/console migration:unsigned_keys
    $ sudo php83 bin/console db:update
    $ sudo php83 bin/console db:check

    ```
- Caso vá utilizar o inventory do glpi para equipamentos sw, nas, e esxi descomente os arquivos .sh  em > install_glpiti > templates.
  - Basta descomentar as linhas desejadas e configurar com seu usuario e senha e commutity para os equipamentos que utiliza SNMP. 
- Após a instalação basta abrir a pagina e seguir a configuração (glpi.yourdomain.com.br):
  - Usuário e senha do banco glpi caso não alterou na role install_mysql:
    - user: infra
    - passwd: P@sswd
  - Passo a passo da instalação após concluir a instalação de todas as roles está no final desse README [GLPI Click aqui](#glpi-instalação).

### Install Zabbix

  - Install Zabbix
    - Atenção:
      - Caso não tenha um servidor DNS configurado para acessar as aplicações adicione em seu computador local.
        - Windows: c:\Windows\System32\drivers\etc\hosts
          - 192.168.135.139 esse é o ip do meu servidor vm-node-02 no momento me que gerei a VM de teste, altere para seu ip
          - 192.168.135.139 zabbix.yourdomain.com.br
          - Caso tenha mais aplicações faça o mesmo
        - Linux: /etc/hosts
          - 192.168.135.139 esse é o ip do meu servidor vm-node-02 no momento me que gerei a VM de teste, altere para seu ip
          - 192.168.135.139 zabbix.yourdomain.com.br
          - Caso tenha mais aplicações faça o mesmo

    - Zabbix
      - Considere ajustar os valores dos arquivos em install_zabbix > files
        - Altere para seus dados de dominio entre outros que achar necessário alterar;
      - Considere ajustar os valores do arquivo em install_zabbix > vars > main.yml
        - Altere para seus dados de dominio entre outros que achar necessário alterar;
      - Após a instalação basta abrir a pagina e seguir a configuração (zabbix.yourdomain.com.br):
        - Passo a passo da instalação após concluir a instalação de todas as roles está no final desse README.
      - Passo a passo da instalação após concluir a instalação de todas as roles está no final desse [ZABBIX Click Aqui](#zabbix-instalação).


### Install Grafana

Passo a passo da instalação após concluir a instalação do grafana está no final desse [GRAFANA Click aqui](#grafana-instalação).

### SNMP
    * Local dos arquivos MIB: /usr/share/snmp/mibs
    * Comando: snmpwalk -v 1 -c public -O e 192.168.0.1
    * Comando: snmpwalk -v 2c -c public -O e 192.168.0.1
    * Comando: snmpwalk -v 3 -c public -O e 192.168.0.1

## ZABBIX Instalação

- Acessar a url [ZABBIX](http://zabbix.yourdomain.com.br/)
- Selecione o Idioma e click em `Next Step`
    ![Passo 01](.github/assets/images/readme/zabbix/install_zabbix_01.jpg)
- Se não tiver faltando nenhum intem obrigatório click em `Next Step`
    ![Passo 02](.github/assets/images/readme/zabbix/install_zabbix_02.jpg)
- Informe os dados de acesso ao banco do zabbix e click em `Next Step`
    ![Passo 03](.github/assets/images/readme/zabbix/install_zabbix_03.jpg)
- Selecione as configurações de acordo com sua preferência e click em `Next Step`
    ![Passo 04](.github/assets/images/readme/zabbix/install_zabbix_04.jpg)
- Revise as informações e click em `Next Step`
    ![Passo 05](.github/assets/images/readme/zabbix/install_zabbix_05.jpg)
- Click em `Finish`
    ![Passo 06](.github/assets/images/readme/zabbix/install_zabbix_06.jpg)
- Acesse o zabbix:
  - Usuário: Admin
  - Senha: zabbix
    ![Passo 07](.github/assets/images/readme/zabbix/install_zabbix_07.jpg)
- Feito todos os passos conseguiu instalar e logar no zabbix com sucesso
    ![Passo 08](.github/assets/images/readme/zabbix/install_zabbix_08.jpg)

### Configurar no frontend de acesso ao zabbix
- Navegar para:
  - Administração > Geral > Outras > Frontend URL
  - Definir sua url de acesso ao zabbix: http://zabbix.yourdomain.com.br/
    ![Passo 09](.github/assets/images/readme/zabbix/install_zabbix_09.jpg)

### Configurar Media Type E-mail

- Navegar para:
  - Alerts > Media Types > Editar o Media Type E-mail
    - Adicionar suas configurações de SMTP de acordo com o seu provedor de e-mail e click em Enable e em Update.
    ![Passo 10](.github/assets/images/readme/zabbix/install_zabbix_10.jpg)
    - Click em Test ao lado do Media Type editado e envie um e-mail de teste.
    ![Passo 11](.github/assets/images/readme/zabbix/install_zabbix_11.jpg)

### Restore Database Zabbix 6.0
- Backup database antigo:
  ```
  sudo mysqldump -u zabbix -p --default-character-set=utf8mb4 --single-transaction --quick zabbix > zabbix60_backup.sql

  ```

- Restore database antigo:
  ```
  sudo systemctl stop zabbix-server
  sudo systemctl stop zabbix-agent
  sudo systemctl stop zabbix-web-server
  sudo tar -xvjf zabbix-2024-02-21-14-25.tar.bz2
  sudo mysql -u root -pP@sswd zabbix < /home/administrator/zabbix_60.sql
  sudo systemctl start zabbix-server
  sudo systemctl start zabbix-agent
  sudo systemctl start zabbix-web-server
  ```

## GRAFANA Instalação
- Acessar a url [GRAFANA](http://grafana.yourdomain.com.br/)
- Informe o usuário e senha padrão:
  - Usuário: admin
  - Senha: admin
    ![Passo 01](.github/assets/images/readme/grafana/install_grafana_01.jpg)
- Alterar a senha padrão
    ![Passo 02](.github/assets/images/readme/grafana/install_grafana_02.jpg)
- Após alterar a senha será direcionado para a página inicial
    ![Passo 03](.github/assets/images/readme/grafana/install_grafana_03.jpg)
- Já foi adicionado o datasource do zabbix, click para editar
    ![Passo 04](.github/assets/images/readme/grafana/install_grafana_04.jpg)
- Vamos criar um dashboard de teste, click no icone em destaque
    ![Passo 05](.github/assets/images/readme/grafana/install_grafana_05.jpg)
- Click novamente no icone em destaque e selecione o datasource do zabbix
    ![Passo 06](.github/assets/images/readme/grafana/install_grafana_06.jpg)
    ![Passo 07](.github/assets/images/readme/grafana/install_grafana_07.jpg)
- Agora nessa página selecione na sequencia de acordo com a numeração da imagem abaixo
    ![Passo 08](.github/assets/images/readme/grafana/install_grafana_08.jpg)
- Caso deseje adicione uma nova query e selecione mais itens que achar conveniente
    ![Passo 09](.github/assets/images/readme/grafana/install_grafana_09.jpg)
- Siga os mesmos passos da imagem anterior
    ![Passo 10](.github/assets/images/readme/grafana/install_grafana_10.jpg)
- Salve o dashboard
    ![Passo 11](.github/assets/images/readme/grafana/install_grafana_11.jpg)
- Agora você pode criar quantos dashboards quiser utilizando os dados do Zabbix.

## GLPI Instalação

- Acessar a url [GLPI](http://glpi.yourdomain.com.br/)
- Selecione o Idioma e click em `OK`
    ![Instalação passo 01](.github/assets/images/readme/glpi/install_glpi_01.jpg)
- Click em `Continuar`
    ![Instalação passo 02](.github/assets/images/readme/glpi/install_glpi_02.jpg)
- Click em `Instalar`
    ![Instalação passo 03](.github/assets/images/readme/glpi/install_glpi_03.jpg)
- Se não tiver faltando nenhum intem obrigatório click em `Continuar`
    ![Instalação passo 04](.github/assets/images/readme/glpi/install_glpi_04.jpg)
- Caso esteja utilizando o banco local siga como na imagem:
  - Servidor SQL: `127.0.0.1`
  - Usuário: `infra`
  - Senha: `P@sswd`
    ![Instalação passo 05](.github/assets/images/readme/glpi/install_glpi_05.jpg)
- Se existir o banco glpi, selecione-o e click em `Continuar`
    ![Instalação passo 06](.github/assets/images/readme/glpi/install_glpi_06.jpg)
- Click em `Continuar`
    ![Instalação passo 07](.github/assets/images/readme/glpi/install_glpi_07.jpg)
- Desmarque `Enviar estatísticas de uso`  e click em `Continuar`
    ![Instalação passo 08](.github/assets/images/readme/glpi/install_glpi_08.jpg)
- Click em `Continuar`
    ![Instalação passo 09](.github/assets/images/readme/glpi/install_glpi_09.jpg)
- Click em `Usar GLPI`
    ![Instalação passo 10](.github/assets/images/readme/glpi/install_glpi_10.jpg)
- Para realizar login utilize o usuário e senha padrão:
  - Usuário: glpi
  - Senha: glpi
    ![Instalação passo 11](.github/assets/images/readme/glpi/install_glpi_11.jpg)

- Habilitar Inventory GLPI
  - Click nos icones em destaque conforme a imagem abaixo:
  ![Instalação passo 12](.github/assets/images/readme/glpi/install_glpi_12.jpg)

  - Aguarde finalizar a instalação e depois habilite o plugin como na imagem abaixo:
  ![Instalação passo 13](.github/assets/images/readme/glpi/install_glpi_13.jpg)

  - Se tudo deu certo ficará como abaixo:
  ![Instalação passo 14](.github/assets/images/readme/glpi/install_glpi_14.jpg)

  - Agora click em GLPI Inventory:
  ![Instalação passo 15](.github/assets/images/readme/glpi/install_glpi_15.jpg)

  - Agora vamos habilitar os módulos em "Módulos de agentes":
  ![Instalação passo 16](.github/assets/images/readme/glpi/install_glpi_16.jpg)
    - Marque o inventário de VMware caso vá utilizar e depois click em atualizar
  ![Instalação passo 17](.github/assets/images/readme/glpi/install_glpi_17.jpg)
    - Marque o inventário de SNPM caso vá utilizar e depois click em atualizar
  ![Instalação passo 18](.github/assets/images/readme/glpi/install_glpi_18.jpg)
    - Para finalizar acesse: Administração > Inventário e habilite o inventário e click em salvar, como descrito na imagem abaixo:
  ![Instalação passo 19](.github/assets/images/readme/glpi/install_glpi_19.jpg)
  - Realizado todos os passos o invetário via script do servidor vai funcionar e para seus computadores locais, basta instalar o glpi-agent e configurar com a sua URL do glpi de inventory para sincronizar em seu GLPI.


- Exemplo Rack
    * Para aparecer as imagens dos ativos, adicione mais informações no modelo de seus itens na lista suspensa
    * Se fizer bem estruturado conseguimos um nível de gestão de inventário muito bom, veja um exemplo: 
    ![Exemplo de Rack GLPI](.github/assets/images/readme/glpi/rack_glpi.jpg)

- Segue abaixo algumas dicas de configuração do GLPI (Vincular ao AD, inventory automatico)
- Na pasta rhel-apache-glpi-mkdocs-php-zabbix\install_glpiti\files
  - Contém o agente linux e windows e um readme de como usar cada um deles.
  - Script inventory para configurar no crontab, deve se alterar o arquivo /var/www/html/glpi/scripts_inventory/inventory.sh com seus dispositivos senhas e strings snmp. (Esse script já está configurado no crontab, porém com todas as linhas comentadas, basta adicionar os dados e salvar ele roda uma vez por semana)
    - Caso deseje basta executar manualmente o /var/www/html/glpi/scripts_inventory/inventory.sh e depois que finalizar o /var/www/html/glpi/scripts_inventory/injector.sh.

- Filtros de conexão para o AD
    * Filtro da conexão: (&(objectClass=user)(objectCategory=person))
    * BaseDN: dc=domain, dc=com, dc=br
    * RootDN (para ligações não anônimas): domain\user
    * Campo de Login: samaccountname

- Filtro de conexão grupos para o AD
    * Tipos de pesquisa: Em grupos
    * Filtrar para pesquisar em grupos: (objectClass=group)

- Editar php.ini
    * Busque pela opção session.cookie_httponly e adicione On ao lado do sinal de =

- Licença Marketplace 
    * Criar conta GLPI Network e vincular chave na pagina de plugin

- Glpi Inventory Nativo
    * https://glpi-agent.readthedocs.io/en/latest/index.html
    * Baixar GLPI Agent
    * Instalar nas máquinas
        * Em Remote Targets cole a URL da pagina invetory de seu ambiente (deixar no final apenas inventory.php)
        * http://glpi.yourdomain.com.br/front/inventory.php
        * acessar na maquina o agente: localhost:62354 (pode forçar o inventory)

- Glpi Inventory Plugin
    * Baixar no site da GLPI Project 
        * Url:  https://github.com/glpi-project/glpi-inventory-plugin/releases/download/1.3.4/glpi-glpiinventory-1.3.4.tar.bz2
        * Eu não adiciono pelo marketplace do GLPI, realizo download direto na pasta do GLPI:
            $ cd /var/www/html/glpi/plugins
            $ wget https://github.com/glpi-project/glpi-inventory-plugin/releases/download/1.3.4/glpi-glpiinventory-1.3.4.tar.bz2
            $ tar -xvf glpi-glpiinventory-1.3.4.tar.bz2
        * Acesse seu GLPI na aba Plugins e click em instalar no plugin adicionado
        * Habilite o plugin
        * Em administração vai aparecer a aba Inventory GLPI
        * para inventariar use os mesmos passos realizados no Inventory Nativo]

        * Na aba Inventário GLPI > click em Geral > Configuração Geral > Módulo dos agentes > habilite os módulos que deseja utilizar

- Glpi Inventory vCenter
    * https://glpi-agent.readthedocs.io/en/latest/man/glpi-esx.html

- Para o script de inventory automatico você deve alterar para seus dispositivos internos ou desativar no crontab.
  - Veja na task de instalação do GLPI e procure pelos scripts /var/www/html/glpi/scripts_inventory/injector.sh e /var/www/html/glpi/scripts_inventory/inventory.sh

## 👨‍💻 Expert

<p>
    <img 
      align=left 
      margin=10 
      width=80 
      src=".github/assets/images/readme/perfil/perfil.jpg"
    />
    <p>&nbsp&nbsp&nbspFelipe M Ferreira<br>
    &nbsp&nbsp&nbsp
    <a href="https://github.com/felipeb2a">
    GitHub</a>&nbsp;|&nbsp;
    <a href="https://www.linkedin.com/in/felipe-marques-ti/">LinkedIn</a>
&nbsp;|&nbsp;
    <a href="https://www.instagram.com/felipeb2a/">
    Instagram</a>
&nbsp;|&nbsp;</p>
</p>
<br/><br/>
<p>

---

⌨️ com 💜 por [Felipe M Ferreira](https://github.com/felipeb2a)