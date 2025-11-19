# HTTPServer_RockyLinux-Manual
Passo a passo da instalação e configuração de um servidor Web usando Rocky Linux e seus derivados

# 🌐 Guia Completo: Servidor Web Apache no Rocky Linux

<div align="center">

![Rocky Linux](https://img.shields.io/badge/Rocky%20Linux-10BE39?style=for-the-badge&logo=rockylinux&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=apache&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

**Manual completo para instalação e configuração de servidor web Apache (httpd)**

[Instalação](#-instalação) • [Configuração](#%EF%B8%8F-configuração-básica) • [Virtual Hosts](#-virtual-hosts) • [SSL/TLS](#-ssltls-com-lets-encrypt)

</div>

---

## 📋 Índice

- [Pré-requisitos](#-pré-requisitos)
- [Instalação](#-instalação)
- [Configuração Básica](#%EF%B8%8F-configuração-básica)
- [Virtual Hosts](#-virtual-hosts)
- [SSL/TLS com Let's Encrypt](#-ssltls-com-lets-encrypt)
- [Módulos do Apache](#-módulos-do-apache)
- [Segurança](#-segurança)
- [Otimização de Performance](#-otimização-de-performance)
- [Logs e Monitoramento](#-logs-e-monitoramento)
- [Troubleshooting](#-troubleshooting)
- [Contribuindo](#-contribuindo)

---

## 🔧 Pré-requisitos

Antes de começar, certifique-se de que você possui:

- Rocky Linux 8/9 ou derivados (AlmaLinux, Oracle Linux)
- Acesso root ou privilégios sudo
- Endereço IP estático (recomendado)
- Nome de domínio apontando para seu servidor (opcional, para SSL)
- Conexão com a internet

---

## 📦 Instalação

### Passo 1: Atualizar o Sistema

Atualize todos os pacotes do sistema:

```bash
sudo dnf update -y
```

### Passo 2: Instalar o Apache

Instale o servidor web Apache (httpd):

```bash
sudo dnf install httpd -y
```

### Passo 3: Verificar a Instalação

Confirme a versão instalada:

```bash
httpd -v
```

### Passo 4: Iniciar e Habilitar o Serviço

Inicie o Apache e configure para iniciar automaticamente:

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

Verifique o status do serviço:

```bash
sudo systemctl status httpd
```

---

## ⚙️ Configuração Básica

### Estrutura de Diretórios

Conheça os principais diretórios do Apache:

```bash
/etc/httpd/                 # Diretório principal de configuração
/etc/httpd/conf/            # Arquivos de configuração principal
/etc/httpd/conf.d/          # Configurações adicionais
/etc/httpd/conf.modules.d/  # Configuração de módulos
/var/www/html/              # Diretório raiz dos sites
/var/log/httpd/             # Logs do Apache
```

### Arquivo de Configuração Principal

O arquivo principal está localizado em:

```bash
sudo nano /etc/httpd/conf/httpd.conf
```

### Criar uma Página de Teste

Crie um arquivo HTML simples:

```bash
sudo nano /var/www/html/index.html
```

Adicione o conteúdo:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bem-vindo ao Apache</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 50px; }
        h1 { color: #D22128; }
    </style>
</head>
<body>
    <h1>🎉 Apache está funcionando!</h1>
    <p>Servidor configurado com sucesso no Rocky Linux</p>
</body>
</html>
```

### Testar Configuração

Verifique se há erros na configuração:

```bash
sudo httpd -t
```

ou

```bash
sudo apachectl configtest
```

---

## 🔒 Configurar o Firewall

### Liberar Portas HTTP e HTTPS

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### Verificar Regras do Firewall

```bash
sudo firewall-cmd --list-all
```

### Testar no Navegador

Acesse no navegador:

```
http://seu-ip-ou-dominio
```

---

## 🏠 Virtual Hosts

Virtual Hosts permitem hospedar múltiplos sites no mesmo servidor.

### Passo 1: Criar Estrutura de Diretórios

```bash
sudo mkdir -p /var/www/site1.com/html
sudo mkdir -p /var/www/site2.com/html
```

### Passo 2: Definir Permissões

```bash
sudo chown -R $USER:$USER /var/www/site1.com/html
sudo chown -R $USER:$USER /var/www/site2.com/html
sudo chmod -R 755 /var/www
```

### Passo 3: Criar Páginas de Teste

Para o site1.com:

```bash
nano /var/www/site1.com/html/index.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>Site 1</title>
</head>
<body>
    <h1>Bem-vindo ao Site1.com</h1>
</body>
</html>
```

Para o site2.com:

```bash
nano /var/www/site2.com/html/index.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>Site 2</title>
</head>
<body>
    <h1>Bem-vindo ao Site2.com</h1>
</body>
</html>
```

### Passo 4: Criar Arquivos de Virtual Host

Crie o arquivo para site1.com:

```bash
sudo nano /etc/httpd/conf.d/site1.com.conf
```

Adicione a configuração:

```apache
<VirtualHost *:80>
    ServerName site1.com
    ServerAlias www.site1.com
    ServerAdmin admin@site1.com
    DocumentRoot /var/www/site1.com/html
    
    <Directory /var/www/site1.com/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog /var/log/httpd/site1.com_error.log
    CustomLog /var/log/httpd/site1.com_access.log combined
</VirtualHost>
```

Crie o arquivo para site2.com:

```bash
sudo nano /etc/httpd/conf.d/site2.com.conf
```

```apache
<VirtualHost *:80>
    ServerName site2.com
    ServerAlias www.site2.com
    ServerAdmin admin@site2.com
    DocumentRoot /var/www/site2.com/html
    
    <Directory /var/www/site2.com/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog /var/log/httpd/site2.com_error.log
    CustomLog /var/log/httpd/site2.com_access.log combined
</VirtualHost>
```

### Passo 5: Testar e Reiniciar

```bash
sudo httpd -t
sudo systemctl restart httpd
```

---

## 🔐 SSL/TLS com Let's Encrypt

### Passo 1: Instalar Certbot

Habilite o repositório EPEL:

```bash
sudo dnf install epel-release -y
```

Instale o Certbot e o plugin do Apache:

```bash
sudo dnf install certbot python3-certbot-apache -y
```

### Passo 2: Obter Certificado SSL

Para um único domínio:

```bash
sudo certbot --apache -d site1.com -d www.site1.com
```

Para múltiplos domínios:

```bash
sudo certbot --apache -d site1.com -d www.site1.com -d site2.com -d www.site2.com
```

### Passo 3: Renovação Automática

Teste a renovação:

```bash
sudo certbot renew --dry-run
```

O Certbot configura automaticamente a renovação via cron/systemd timer.

Verificar timer de renovação:

```bash
sudo systemctl status certbot-renew.timer
```

### Passo 4: Verificar Certificados

Liste os certificados instalados:

```bash
sudo certbot certificates
```

## 🔍 Troubleshooting

### Problema: Apache não inicia

**Solução:** Verifique os logs de erro:

```bash
sudo systemctl status httpd -l
sudo journalctl -xe -u httpd
```

Teste a configuração:

```bash
sudo httpd -t
```

### Problema: Erro 403 Forbidden

**Solução:** Verifique permissões:

```bash
sudo ls -la /var/www/html/
sudo chmod -R 755 /var/www/html/
```

Verifique o SELinux:

```bash
sudo ls -Z /var/www/html/
sudo restorecon -Rv /var/www/html/
```

### Problema: Site não carrega (timeout)

**Solução:** Verifique o firewall:

```bash
sudo firewall-cmd --list-all
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

Verifique se o Apache está escutando:

```bash
sudo ss -tulnp | grep :80
sudo netstat -tulnp | grep :80
```

### Problema: Virtual Host não funciona

**Solução:** Verifique a sintaxe do arquivo:

```bash
sudo httpd -t
```

Liste os Virtual Hosts configurados:

```bash
sudo httpd -S
```

ou

```bash
sudo apachectl -S
```

### Reiniciar ou Recarregar

Recarregar configuração (sem derrubar conexões):

```bash
sudo systemctl reload httpd
```

Reiniciar completamente:

```bash
sudo systemctl restart httpd
```

---

## 📈 Comandos Úteis

### Status e Controle

```bash
# Verificar status
sudo systemctl status httpd

# Iniciar serviço
sudo systemctl start httpd

# Parar serviço
sudo systemctl stop httpd

# Reiniciar serviço
sudo systemctl restart httpd

# Recarregar configuração
sudo systemctl reload httpd

# Habilitar na inicialização
sudo systemctl enable httpd

# Desabilitar na inicialização
sudo systemctl disable httpd
```

### Diagnóstico

```bash
# Testar configuração
sudo apachectl configtest

# Listar módulos carregados
httpd -M

# Listar Virtual Hosts
httpd -S

# Verificar versão
httpd -v

# Informações completas
httpd -V
```

---

## 📚 Recursos Adicionais

- [Documentação Oficial do Apache](https://httpd.apache.org/docs/)
- [Rocky Linux Documentation](https://docs.rockylinux.org/)
- [Let's Encrypt](https://letsencrypt.org/)
- [Apache Security Tips](https://httpd.apache.org/docs/2.4/misc/security_tips.html)

---

## ✍️ Autor

Criado por jovictornm para a comunidade Linux

**Se este guia foi útil, deixe uma ⭐ no repositório!**

---

<div align="center">

**[⬆ Voltar ao topo](#-guia-completo-servidor-web-apache-no-rocky-linux)**

</div>
