# Automated Server Provisioning & Configuration Management with Ansible
[200~![Ansible](https://img.shields.io/badge/Ansible-E00?style=for-the-badge&logo=ansible&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Linux](https://img.shields.io/badge/Fedora-51A2DA?style=for-the-badge&logo=fedora&logoColor=white)

Solución integral de **Infraestructura como Código (IaC)** y gestión de configuración para aprovisionar, securizar y desplegar servicios sobre servidores Linux (Ubuntu Server) de forma modular, automatizada e idempotente utilizando **Ansible**.

---
## Descripción General
Este proyecto automatiza el ciclo de vida completo de preparación de un servidor en la nube desde una estación de control local (Fedora / Linux). Reemplaza la configuración manual mediante SSH por una arquitectura declarativa basada en **Roles**.

### Características principales
* **Actualización y Seguridad:** Actualización completa de paquetes (`apt upgrade`), herramientas de diagnóstico (`curl`, `htop`, `vim`, `git`) y protección contra ataques de fuerza bruta con `fail2ban`.
* **Gestión de Accesos (SSH):** Inyección automatizada y segura de llaves SSH autorizadas.
* **Servidor Web:** Instalación y aseguramiento del demonio `nginx` mediante Systemd.
* **Despliegue de Aplicación:** Transferencia y extracción desasistida de un paquete comprimido (`.tar.gz`) con los permisos requeridos (`www-data`).
* **Ejecución Selectiva:** Uso de etiquetas (`tags`) para ejecutar únicamente tareas puntuales sin reprocesar todo el sistema.

---
## Arquitectura de Roles
El proyecto sigue la convención estándar de diseño modular de Ansible:

| Rol | Tag | Descripción / Módulos Principales |
| :--- | :--- | :--- |
| **`base`** | `base` | Actualización de caché del sistema, instalación de utilidades y habilitación de `fail2ban`. |
| **`ssh`** | `ssh` | Despliegue de llaves públicas con el módulo nativo `authorized_key`. |
| **`nginx`** | `nginx` | Instalación del binario Nginx y aseguramiento de estados (`started`, `enabled`). |
| **`app`** | `app` | Despliegue del sitio web mediante el módulo `unarchive` en `/var/www/html`. |

---
## Estructura del Repositorio

```text
.
├── inventory.ini.example   # Plantilla pública de inventario (sin datos sensibles)
├── setup.yml               # Playbook maestro que orquesta los roles
├── .gitignore              # Exclusión de claves privadas (.pem) e inventarios reales
├── DOCUMENTATION.md        # Análisis técnico detallado de conceptos
└── roles/
    ├── base/
    │   └── tasks/main.yml
    ├── ssh/
    │   ├── files/dummy_key.pub
    │   └── tasks/main.yml
    ├── nginx/
    │   └── tasks/main.yml
    └── app/
        ├── files/website.tar.gz
        └── tasks/main.yml
Requisitos Previos
Nodo de Control (Local):

Linux (Fedora, Debian/Ubuntu, CentOS, etc.)

Ansible >= 2.15 (sudo dnf install ansible o sudo apt install ansible)

Python 3 instalado

Nodo Administrado (Remoto):

Servidor accesible vía SSH (ej. AWS EC2, DigitalOcean) con Python 3 instalado.

Usuario con privilegios sudo sin contraseña.
Guía de Inicio Rápido
1. Clonar el repositorio
Bash
git clone [https://github.com/](https://github.com/)<TU_USUARIO>/ansible-server-automation.git
cd ansible-server-automation
2. Configurar el inventario
Copia la plantilla inventory.ini.example a inventory.ini:

Bash
cp inventory.ini.example inventory.ini
nano inventory.ini
Modifica el archivo con la IP pública de tu servidor y la ruta local a tu llave privada:

Ini, TOML
[webservers]
<IP_DE_TU_SERVIDOR> ansible_user=ubuntu ansible_ssh_private_key_file=/ruta/a/tu/clave.pem

[webservers:vars]
ansible_python_interpreter=/usr/bin/python3
3. Validar conectividad (Ad-hoc Ping)
Verifica la comunicación SSH antes de ejecutar el playbook:

Bash
ansible -i inventory.ini webservers -m ping
Salida esperada: Respuesta "ping": "pong" en color verde.
Modos de Ejecución
1. Aprovisionamiento Completo
Ejecuta todos los roles en secuencia:

Bash
ansible-playbook -i inventory.ini setup.yml
2. Ejecución Selectiva por Tags
Para ejecutar únicamente roles específicos:

Bash
# Desplegar únicamente cambios en la aplicación web
ansible-playbook -i inventory.ini setup.yml --tags "app"

# Ejecutar tareas relacionadas con Nginx y llaves SSH
ansible-playbook -i inventory.ini setup.yml --tags "nginx,ssh"
3. Modo Dry-Run (Verificación sin aplicar cambios)
Evalúa el comportamiento antes de alterar el estado real del servidor:

Bash
ansible-playbook -i inventory.ini setup.yml --check
Seguridad y Buenas Prácticas
Control de secretos: Archivos con extensión *.pem, *.key y el inventario real inventory.ini están declarados en .gitignore para evitar filtraciones de credenciales.

Idempotencia: La infraestructura se declara como estado deseado. Corridas consecutivas generan changed=0 sin romper configuraciones activas.

Separación de responsabilidades: Tareas aisladas en roles independientes y reutilizables.
