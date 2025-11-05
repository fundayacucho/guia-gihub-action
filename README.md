# 🚀 Guía para configurar GitHub Actions para despliegue automático

Esta guía te ayudará a configurar un workflow de GitHub Actions que se activa al hacer `push` a una rama específica (por ejemplo, `main`) y ejecuta un script de despliegue en tu servidor vía SSH.

---

## 🧱 Requisitos previos

1. **Servidor con acceso SSH** (CentOS, AlmaLinux, Ubuntu, etc.)
2. **Usuario de despliegue** en el servidor (ej. `deployuser`)
3. **Llave SSH configurada**:
   - Llave privada guardada como secreto en GitHub
   - Llave pública agregada a `~/.ssh/authorized_keys` del usuario en el servidor
4. **Script de despliegue** en el servidor (ej. `/home/deployuser/scripts/deploy.sh`)
5. **Permisos correctos** en `.ssh`, `authorized_keys`, y el script

---

## 🔐 Paso 1: Crear y registrar la llave SSH

### En tu máquina local o servidor:

```bash
ssh-keygen -t ed25519 -C "github-actions-deploy"
Llave privada: ~/.ssh/id_ed25519

Llave pública: ~/.ssh/id_ed25519.pub

En el servidor:
bash
mkdir -p ~/.ssh
cat id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R deployuser:deployuser ~/.ssh
En GitHub:
Ve a Settings → Secrets → Actions

Crea un nuevo secreto:

Name: VPS_SSH_PRIVATE_KEY

Value: contenido completo de id_ed25519

⚙️ Paso 2: Crear el archivo del workflow
Crea el archivo .github/workflows/deploy.yml en tu repositorio:

yaml
name: Deploy to VPS

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
🧪 Paso 3: Probar el workflow
Haz git push a la rama main

Ve a la pestaña Actions en GitHub

Verifica que el workflow se ejecuta correctamente

Revisa los logs en el servidor (ej. /home/deployuser/deploy.log)

🧠 Consejos adicionales
Usa npx o rutas absolutas en el script para evitar conflictos de versiones de Node/npm

Asegura que el script tenga permisos de ejecución: chmod +x deploy.sh

Puedes agregar más secretos si tu script necesita tokens, variables, etc.

Si usas SELinux, aplica restorecon y semanage fcontext para que Apache/Nginx pueda leer los archivos
