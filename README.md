Automated Server Provisioning & Configuration Management with Ansible

![Ansible](https://img.shields.io/badge/Ansible-E00?style=for-the-badge&logo=ansible&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Fedora](https://img.shields.io/badge/Fedora-51A2DA?style=for-the-badge&logo=fedora&logoColor=white)

Solución integral de Infraestructura como Código (IaC) y gestión de configuración para aprovisionar, securizar y desplegar servicios sobre servidores Linux (Ubuntu Server) de forma modular, automatizada e idempotente utilizando Ansible.

Descripción General

Este proyecto automatiza el ciclo de vida completo de preparación de un servidor en la nube desde una estación de control local (Fedora / Linux).

Reemplaza la configuración manual mediante SSH por una arquitectura declarativa basada en Roles de Ansible, permitiendo automatizar tareas de instalación, configuración, seguridad y despliegue de aplicaciones.

Características principales
Actualización y Seguridad: actualización completa de paquetes mediante apt upgrade, instalación de herramientas de diagnóstico como curl, htop, vim y git, además de protección contra ataques de fuerza bruta mediante fail2ban.
Gestión de Accesos SSH: inyección automatizada y segura de llaves SSH autorizadas mediante el módulo authorized_key.
Servidor Web: instalación, configuración y administración del servicio nginx mediante Systemd.
Despliegue de Aplicación: transferencia y extracción automatizada de un paquete comprimido .tar.gz en /var/www/html, asignando los permisos correspondientes al usuario www-data.
Ejecución Selectiva: utilización de tags para ejecutar únicamente determinadas tareas sin necesidad de reprocesar toda la configuración del servidor.
Idempotencia: las tareas están diseñadas para que puedan ejecutarse múltiples veces sin generar cambios innecesarios en el servidor.
Arquitectura de Roles

El proyecto sigue una arquitectura modular basada en Roles de Ansible.

Rol	Tag	Descripción	Módulos Principales
base	base	Actualización del sistema, instalación de utilidades y configuración de seguridad.	apt, service
ssh	ssh	Configuración de llaves públicas autorizadas para acceso SSH.	authorized_key
nginx	nginx	Instalación y administración del servidor web Nginx.	apt, service
app	app	Despliegue de la aplicación web en /var/www/html.	unarchive, file
Estructura del Repositorio
.
├── inventory.ini.example       # Plantilla pública del inventario
├── setup.yml                   # Playbook principal
├── .gitignore                  # Exclusión de archivos sensibles
├── DOCUMENTATION.md            # Documentación técnica
└── roles/
    ├── base/
    │   └── tasks/
    │       └── main.yml
    │
    ├── ssh/
    │   ├── files/
    │   │   └── dummy_key.pub
    │   └── tasks/
    │       └── main.yml
    │
    ├── nginx/
    │   └── tasks/
    │       └── main.yml
    │
    └── app/
        ├── files/
        │   └── website.tar.gz
        └── tasks/
            └── main.yml
Requisitos Previos
Nodo de Control

El nodo de control es el equipo desde donde se ejecutará Ansible.

Requisitos:

Sistema operativo Linux.
Fedora, Debian, Ubuntu, CentOS u otra distribución compatible.
Ansible >= 2.15.
Python 3.
Acceso SSH al servidor remoto.

Para instalar Ansible en Fedora:

sudo dnf install ansible

En Debian o Ubuntu:

sudo apt install ansible

Verificar la instalación:

ansible --version
Nodo Administrado

El nodo administrado es el servidor remoto que será configurado.

Requisitos:

Ubuntu Server.
Acceso mediante SSH.
Python 3 instalado.
Usuario con privilegios sudo.
Acceso a Internet para descargar paquetes.
Llave privada SSH válida.

El servidor puede encontrarse en proveedores como AWS EC2, DigitalOcean, Contabo u otros servicios de infraestructura cloud.

Guía de Inicio Rápido
1. Clonar el repositorio

Clona el proyecto desde GitHub:

git clone https://github.com/<TU_USUARIO>/ansible-server-automation.git

Ingresa al directorio:

cd ansible-server-automation
2. Configurar el Inventario

Copia el archivo de inventario de ejemplo:

cp inventory.ini.example inventory.ini

Edita el archivo:

nano inventory.ini

Configura la IP pública de tu servidor y la ruta de tu llave privada:

[webservers]
<IP_DE_TU_SERVIDOR> ansible_user=ubuntu ansible_ssh_private_key_file=/ruta/a/tu/clave.pem

[webservers:vars]
ansible_python_interpreter=/usr/bin/python3

Por ejemplo:

[webservers]
203.0.113.10 ansible_user=ubuntu ansible_ssh_private_key_file=/home/user/.ssh/server.pem

[webservers:vars]
ansible_python_interpreter=/usr/bin/python3

No debes subir inventory.ini ni las llaves privadas al repositorio.

3. Validar la Conectividad

Antes de ejecutar el playbook, verifica que Ansible pueda comunicarse correctamente con el servidor remoto.

Ejecuta:

ansible -i inventory.ini webservers -m ping

Si la configuración es correcta, deberías obtener una respuesta similar a:

203.0.113.10 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

La respuesta "ping": "pong" confirma que Ansible puede conectarse correctamente al servidor mediante SSH.

Modos de Ejecución
1. Aprovisionamiento Completo

Para ejecutar todos los roles definidos en el proyecto:

ansible-playbook -i inventory.ini setup.yml

Esto ejecutará las tareas correspondientes a:

Configuración base del servidor.
Instalación de herramientas.
Configuración de seguridad.
Configuración de acceso SSH.
Instalación de Nginx.
Despliegue de la aplicación.
2. Ejecución Selectiva mediante Tags

Los tags permiten ejecutar únicamente determinadas tareas del playbook.

Desplegar únicamente la aplicación
ansible-playbook -i inventory.ini setup.yml --tags "app"
Ejecutar solamente Nginx
ansible-playbook -i inventory.ini setup.yml --tags "nginx"
Ejecutar configuración SSH
ansible-playbook -i inventory.ini setup.yml --tags "ssh"
Ejecutar Nginx y SSH
ansible-playbook -i inventory.ini setup.yml --tags "nginx,ssh"
Ejecutar la configuración base
ansible-playbook -i inventory.ini setup.yml --tags "base"
3. Modo Dry-Run

Ansible permite comprobar qué cambios realizaría sobre el servidor sin aplicarlos realmente.

Para ello:

ansible-playbook -i inventory.ini setup.yml --check

Este modo resulta útil para validar el comportamiento del playbook antes de realizar modificaciones sobre el servidor de producción.

Seguridad y Buenas Prácticas
Control de Secretos

Los archivos que pueden contener información sensible deben mantenerse fuera del repositorio.

El archivo .gitignore debe incluir elementos como:

# Inventarios reales
inventory.ini

# Llaves privadas
*.pem
*.key
*.p12

# Variables sensibles
*.secret
.env

De esta manera se evita subir accidentalmente credenciales, llaves privadas o información relacionada con la infraestructura.

Uso de Inventarios de Ejemplo

El repositorio incluye:

inventory.ini.example

Este archivo funciona como plantilla para que otros usuarios puedan configurar su propia infraestructura sin exponer información sensible.

El inventario real:

inventory.ini

debe permanecer excluido mediante .gitignore.

Idempotencia

Uno de los principios fundamentales de Ansible es la idempotencia.

Esto significa que las tareas pueden ejecutarse varias veces y Ansible solamente realizará cambios cuando sean necesarios.

Por ejemplo:

- name: Instalar Nginx
  apt:
    name: nginx
    state: present

Si Nginx ya está instalado, Ansible no volverá a instalarlo.

De esta forma se consigue una configuración predecible y reproducible.

Despliegue de la Aplicación

El rol app utiliza el módulo unarchive para desplegar un archivo comprimido:

website.tar.gz

El contenido se extrae dentro de:

/var/www/html

Posteriormente se establecen los permisos correspondientes para que Nginx pueda servir correctamente los archivos.

La estructura esperada es:

/var/www/html
├── index.html
├── css/
├── js/
└── assets/
Gestión de Nginx

El rol nginx se encarga de:

Instalar Nginx.
Iniciar el servicio.
Habilitar el servicio para que arranque automáticamente.
Mantener el estado deseado del servidor web.

Conceptualmente, el estado esperado es:

Nginx
├── Instalado
├── Ejecutándose
└── Habilitado al iniciar el sistema
Gestión de Seguridad

El rol base instala y configura herramientas necesarias para la administración del servidor.

Entre ellas:

curl
htop
vim
git
fail2ban

fail2ban ayuda a proteger servicios expuestos contra intentos repetidos de autenticación, especialmente ataques de fuerza bruta sobre SSH.

Flujo de Automatización

El proceso completo puede representarse de la siguiente manera:

                Nodo de Control
                 Fedora / Linux
                       |
                       |
                    Ansible
                       |
                       v
              inventory.ini
                       |
                       v
                Servidor Ubuntu
                       |
          +------------+------------+
          |            |            |
          v            v            v
        base          ssh         nginx
          |            |            |
          |            |            |
          +------------+------------+
                       |
                       v
                     app
                       |
                       v
              /var/www/html
                       |
                       v
                  Aplicación Web
Tecnologías Utilizadas
Tecnología	Uso
Ansible	Automatización y gestión de configuración
Ubuntu Server	Sistema operativo del servidor
Fedora Linux	Nodo de control
Nginx	Servidor web
SSH	Comunicación segura con el servidor
Systemd	Administración de servicios
Fail2ban	Protección contra ataques de fuerza bruta
Git	Control de versiones
IaC	Infraestructura como Código
Objetivos del Proyecto

Este proyecto busca demostrar conocimientos prácticos en:

Infraestructura como Código.
Automatización de servidores.
Administración Linux.
Administración remota mediante SSH.
Ansible.
Arquitectura basada en Roles.
Idempotencia.
Gestión de servicios mediante Systemd.
Seguridad básica de servidores.
Despliegue automatizado de aplicaciones.
Uso de inventarios.
Uso de tags.
Buenas prácticas de Git y gestión de secretos.
Comandos Principales
Comprobar versión de Ansible
ansible --version
Comprobar conectividad
ansible -i inventory.ini webservers -m ping
Ejecutar todo el proyecto
ansible-playbook -i inventory.ini setup.yml
Ejecutar solamente la aplicación
ansible-playbook -i inventory.ini setup.yml --tags "app"
Ejecutar Nginx y SSH
ansible-playbook -i inventory.ini setup.yml --tags "nginx,ssh"
Ejecutar en modo comprobación
ansible-playbook -i inventory.ini setup.yml --check
Resultado Esperado

Después de ejecutar correctamente el playbook, el servidor Ubuntu deberá contar con:

Servidor Ubuntu
│
├── Sistema actualizado
├── Herramientas de administración instaladas
├── Fail2ban habilitado
├── Acceso SSH configurado
├── Nginx instalado
├── Nginx ejecutándose
├── Nginx habilitado al iniciar
└── Aplicación desplegada
        │
        └── /var/www/html

El objetivo final es disponer de un servidor preparado y configurado automáticamente mediante Ansible, reduciendo la intervención manual y permitiendo repetir el proceso de forma consistente sobre diferentes servidores.

Licencia

Este proyecto puede utilizarse con fines educativos y demostrativos para practicar conceptos de DevOps, Linux, Ansible e Infraestructura como Código.
