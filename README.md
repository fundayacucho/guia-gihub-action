# 🚀 Guía completa para configurar GitHub Actions con despliegue automático en CentOS

Esta guía te permite configurar un workflow de GitHub Actions que se activa al hacer `push` a una rama (ej. `main`) y ejecuta un script en tu servidor CentOS vía SSH.

---

## 🧱 Requisitos previos

- Servidor CentOS con acceso SSH
- Usuario `deployuser` con permisos limitados
- Llave SSH configurada entre GitHub y el servidor
- Script de despliegue (`deploy.sh`) en el servidor
- Permisos correctos en carpetas, llaves y SELinux

---

## 👤 Paso 1: Crear el usuario de despliegue

```bash
# como root
useradd -m -s /bin/bash deployuser
passwd -l deployuser  # opcional: deshabilitar contraseña
usermod -aG wheel deployuser  # opcional: acceso sudo limitado
🔐 Paso 2: Configurar la llave SSH
Generar llave (local o en el servidor)
bash
ssh-keygen -t ed25519 -C "github-actions-deploy"
Llave privada: ~/.ssh/id_ed25519

Llave pública: ~/.ssh/id_ed25519.pub

Registrar la llave pública en el servidor
bash
mkdir -p /home/deployuser/.ssh
cat id_ed25519.pub >> /home/deployuser/.ssh/authorized_keys
chown -R deployuser:deployuser /home/deployuser/.ssh
chmod 700 /home/deployuser/.ssh
chmod 600 /home/deployuser/.ssh/authorized_keys
Restaurar contexto SELinux
bash
restorecon -Rv /home/deployuser/.ssh
📁 Paso 3: Crear estructura de carpetas
bash
mkdir -p /home/deployuser/scripts
mkdir -p /home/deployuser/repos
mkdir -p /home/deployuser/build_backups
chown -R deployuser:deployuser /home/deployuser
chmod 755 /home/deployuser
chmod 700 /home/deployuser/.ssh
chmod 750 /home/deployuser/scripts
chmod 755 /home/deployuser/repos
chmod 755 /home/deployuser/build_backups
🧾 Paso 4: Crear el script deploy.sh
Guarda el script en /home/deployuser/scripts/deploy.sh y hazlo ejecutable:

bash
chmod +x /home/deployuser/scripts/deploy.sh
chown deployuser:deployuser /home/deployuser/scripts/deploy.sh
🌐 Paso 5: Configurar GitHub Secrets
En tu repositorio GitHub:

Ve a Settings → Secrets → Actions

Crea un nuevo secreto:

Name: VPS_SSH_PRIVATE_KEY

Value: contenido completo de id_ed25519

⚙️ Paso 6: Crear el workflow .github/workflows/deploy.yml
yaml
name: Deploy to CentOS VPS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup SSH key
        uses: webfactory/ssh-agent@v0.5.4
        with:
          ssh-private-key: ${{ secrets.VPS_SSH_PRIVATE_KEY }}

      - name: Add VPS to known_hosts
        run: |
          mkdir -p ~/.ssh
          ssh-keyscan -H 200.6.157.90 >> ~/.ssh/known_hosts

      - name: Run deploy script on VPS
        run: |
          ssh deployuser@200.6.157.90 "bash -l -c '/home/deployuser/scripts/deploy.sh'"
🧪 Paso 7: Verificar conexión SSH
bash
# desde tu máquina local
ssh -i ~/.ssh/id_ed25519 deployuser@200.6.157.90
🛡️ Paso 8: Ajustes de SELinux y Apache/Nginx
Si usas /home/fundaya/becarios/public_html como DocumentRoot:

bash
dnf install -y policycoreutils-python-utils

semanage fcontext -a -t httpd_sys_content_t "/home/fundaya/becarios/public_html(/.*)?"
restorecon -Rv /home/fundaya/becarios/public_html
🔥 Paso 9: Abrir puertos en el firewall
bash
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
