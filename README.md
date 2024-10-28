Servidor Web Automatizado com Monitoramento e CI/CD

Descrição do Projeto
Este projeto configura automaticamente um servidor web Nginx, com monitoramento de recursos e alertas via Zabbix, backups automáticos e um pipeline de CI/CD para deploy em Docker. O objetivo é garantir a alta disponibilidade de um serviço web essencial para uma startup de tecnologia, com um ambiente de produção facilmente escalável e monitorado.

Requisitos do Sistema
Configuração de um Serviço Web
Configuração automática do servidor web Nginx com uma página padrão.
Monitoramento e Notificação
Integração do servidor com Zabbix para monitorar CPU, memória e disponibilidade do serviço.
Configuração de triggers no Zabbix para alertas automáticos por e-mail ou outro canal.
Backup Automatizado
Script de backup dos arquivos de configuração e logs do Nginx e Zabbix.
Backups diários armazenados com rotação, mantendo apenas os últimos 7 dias.
Pipeline de CI/CD Integrado
Pipeline de CI/CD configurado com GitHub Actions para:
Buildar a imagem Docker.
Realizar testes de saúde no container.
Deploy em ambiente de produção ou teste.
Estrutura do Projeto
Dockerfile: Arquivo de configuração Docker para build da imagem Nginx.
.github/workflows/docker_test.yml: Workflow do GitHub Actions para CI/CD automatizado.
zabbix_agentd.conf: Configuração do agente Zabbix para monitoramento.
backup.sh: Script de backup automático com rotação.
hosts.ini: Configurações de inventário para o Ansible.
Pré-requisitos
Windows com WSL2 e Ubuntu 24.04 LTS
Ansible para automação.
Docker instalado para containers.
GitHub com repositório configurado e permissões de Actions.
Instalando Dependências no WSL (Ubuntu)
Atualize o Ubuntu:

sudo apt update && sudo apt upgrade -y
Instale o Ansible:
-------------------------
sudo apt install ansible -y
Instale o Zabbix Agent:

wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-2+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_6.0-2+ubuntu24.04_all.deb
sudo apt update
sudo apt install zabbix-agent -y

Instale o Docker:

sudo apt install docker.io -y
sudo usermod -aG docker $USER
Configuração do Ansible e Scripts
1. Configurar o Inventário do Ansible

Crie o arquivo hosts.ini na pasta servidor_web com o IP do servidor:
[meu_servidor]
172.31.197.239 ansible_user=ruiva ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_become=true
2. Playbook do Ansible para Instalação do Nginx e Configuração do Zabbix
No arquivo playbook.yml, adicione:

- name: Configurar servidor web com Nginx e Zabbix
  hosts: meu_servidor
  tasks:
    - name: Instalar Nginx
      apt:
        name: nginx
        state: present

    - name: Copiar configuração do Zabbix
      copy:
        src: ./zabbix_agentd.conf
        dest: /etc/zabbix/zabbix_agentd.conf

    - name: Iniciar e habilitar Nginx
      service:
        name: nginx
        state: started
        enabled: true
3. Configuração do Backup Automático
No diretório servidor_web, crie o arquivo backup.sh com o conteúdo:


#!/bin/bash
backup_dir="/backup_nginx"
mkdir -p "$backup_dir"
tar -czf "$backup_dir/backup_$(date +%F).tar.gz" /etc/nginx /var/log/nginx
find "$backup_dir" -type f -mtime +7 -exec rm -f {} \;
Defina permissões e configure o crontab para executar o backup:
chmod +x backup.sh
(crontab -l ; echo "0 0 * * * /caminho/para/backup.sh") | crontab -
Configuração do Pipeline de CI/CD com GitHub Actions
Crie .github/workflows/docker_test.yml:
name: Test Dockerfile

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Build Docker image
        run: docker build . -t servidor_web_test

      - name: Run Docker container
        run: docker run -d -p 80:80 servidor_web_test

      - name: Test Docker container
        run: |
          sleep 5
          curl -I http://localhost:80 | grep "200 OK"
Como Executar o Projeto
Executar o Playbook do Ansible:

bash
ansible-playbook -i hosts.ini playbook.yml
Monitorar os Backups:

Verifique o diretório /backup_nginx para confirmações de backup e rotação diária.
Executar o Pipeline de CI/CD:
