# Automated Server Provisioning & Configuration Management with Ansible

![Ansible](https://img.shields.io/badge/Ansible-E00?style=for-the-badge&logo=ansible&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-51A2DA?style=for-the-badge&logo=linux&logoColor=white)

Solución integral de **Infraestructura como Código (IaC)** y gestión de configuración para aprovisionar, securizar y desplegar servicios sobre servidores Linux (Ubuntu Server) de forma modular, automatizada e idempotente utilizando **Ansible**.

---

## Descripción General

Este proyecto automatiza el ciclo de vida completo de preparación de un servidor en la nube desde una estación de control local (Fedora / Linux).

Reemplaza la configuración manual mediante SSH por una arquitectura declarativa basada en **Roles de Ansible**.

### Características principales

- **Actualización y Seguridad:** actualización completa de paquetes (`apt upgrade`), herramientas de diagnóstico (`curl`, `htop`, `vim`, `git`) y protección contra ataques de fuerza bruta con `fail2ban`.
- **Gestión de Accesos SSH:** inyección automatizada y segura de llaves SSH autorizadas mediante el módulo nativo `authorized_key`.
- **Servidor Web:** instalación y aseguramiento del servicio `nginx` mediante Systemd.
- **Despliegue de Aplicación:** transferencia y extracción automatizada de un paquete comprimido (`.tar.gz`) con los permisos requeridos (`www-data`).
- **Ejecución Selectiva:** uso de etiquetas (`tags`) para ejecutar únicamente tareas puntuales sin reprocesar todo el sistema.
- **Idempotencia:** las tareas pueden ejecutarse múltiples veces sin generar cambios innecesarios cuando el servidor ya se encuentra en el estado deseado.

---

## Arquitectura de Roles

El proyecto sigue la convención estándar de diseño modular de Ansible:

| Rol | Tag | Descripción | Módulos Principales |
| :--- | :--- | :--- | :--- |
| `base` | `base` | Actualización de caché del sistema, instalación de utilidades y habilitación de `fail2ban`. | `apt`, `service` |
| `ssh` | `ssh` | Despliegue de llaves públicas mediante `authorized_key`. | `authorized_key` |
| `nginx` | `nginx` | Instalación del servidor web y aseguramiento de su estado. | `apt`, `service` |
| `app` | `app` | Despliegue del sitio web mediante `unarchive` en `/var/www/html`. | `unarchive`, `file` |

---

## Estructura del Repositorio

```text
.
├── inventory.ini.example
├── setup.yml
├── .gitignore
├── DOCUMENTATION.md
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
```

---

## Requisitos Previos

### Nodo de Control

El nodo de control es el equipo desde donde se ejecutará Ansible.

Requisitos:

- Linux (Fedora, Debian, Ubuntu, CentOS, etc.)
- Ansible >= 2.15
- Python 3 instalado
- Acceso SSH al servidor remoto

Para instalar Ansible en Fedora:

```bash
sudo dnf install ansible
```

En Debian o Ubuntu:

```bash
sudo apt install ansible
```

Verificar la instalación:

```bash
ansible --version
```

### Nodo Administrado

El nodo administrado es el servidor remoto que será configurado.

Requisitos:

- Ubuntu Server
- Acceso mediante SSH
- Python 3 instalado
- Usuario con privilegios `sudo`
- Acceso a Internet para descargar paquetes

El servidor puede encontrarse en proveedores cloud como AWS EC2, DigitalOcean, Contabo u otros.

---

# Guía de Inicio Rápido

## 1. Clonar el repositorio

Clona el proyecto desde GitHub:

```bash
git clone https://github.com/<TU_USUARIO>/ansible-server-automation.git
```

Ingresa al directorio del proyecto:

```bash
cd ansible-server-automation
```

---

## 2. Configurar el Inventario

Copia el archivo de inventario de ejemplo:

```bash
cp inventory.ini.example inventory.ini
```

Edita el archivo:

```bash
nano inventory.ini
```

Configura la IP pública del servidor y la ruta de la llave privada:

```ini
[webservers]
<IP_DE_TU_SERVIDOR> ansible_user=ubuntu ansible_ssh_private_key_file=/ruta/a/tu/clave.pem

[webservers:vars]
ansible_python_interpreter=/usr/bin/python3
```

Ejemplo:

```ini
[webservers]
203.0.113.10 ansible_user=ubuntu ansible_ssh_private_key_file=/home/user/.ssh/server.pem

[webservers:vars]
ansible_python_interpreter=/usr/bin/python3
```

> No debes subir `inventory.ini` ni las llaves privadas al repositorio.

---

## 3. Validar la Conectividad

Antes de ejecutar el playbook, verifica que Ansible pueda comunicarse correctamente con el servidor remoto:

```bash
ansible -i inventory.ini webservers -m ping
```

Salida esperada:

```text
203.0.113.10 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

La respuesta `"ping": "pong"` confirma que Ansible puede conectarse correctamente al servidor mediante SSH.

---

# Modos de Ejecución

## 1. Aprovisionamiento Completo

Ejecuta todos los roles en secuencia:

```bash
ansible-playbook -i inventory.ini setup.yml
```

El proceso realizará las siguientes tareas:

1. Configuración base del servidor.
2. Actualización de paquetes.
3. Instalación de herramientas.
4. Configuración de seguridad.
5. Configuración de acceso SSH.
6. Instalación de Nginx.
7. Despliegue de la aplicación.

---

## 2. Ejecución Selectiva por Tags

Los `tags` permiten ejecutar únicamente determinadas tareas del playbook.

### Desplegar únicamente la aplicación web

```bash
ansible-playbook -i inventory.ini setup.yml --tags "app"
```

### Ejecutar únicamente Nginx

```bash
ansible-playbook -i inventory.ini setup.yml --tags "nginx"
```

### Ejecutar únicamente SSH

```bash
ansible-playbook -i inventory.ini setup.yml --tags "ssh"
```

### Ejecutar Nginx y SSH

```bash
ansible-playbook -i inventory.ini setup.yml --tags "nginx,ssh"
```

### Ejecutar la configuración base

```bash
ansible-playbook -i inventory.ini setup.yml --tags "base"
```

---

## 3. Modo Dry-Run

Ansible permite comprobar qué cambios realizaría sobre el servidor sin aplicarlos realmente.

```bash
ansible-playbook -i inventory.ini setup.yml --check
```

Este modo permite validar el comportamiento del playbook antes de realizar modificaciones sobre el servidor.

---

# Seguridad y Buenas Prácticas

## Control de Secretos

Los archivos que pueden contener información sensible deben mantenerse fuera del repositorio.

El archivo `.gitignore` debe incluir elementos como:

```gitignore
# Inventarios reales
inventory.ini

# Llaves privadas
*.pem
*.key
*.p12

# Archivos con variables sensibles
.env
*.secret
```

Esto evita subir accidentalmente credenciales, llaves privadas o información relacionada con la infraestructura.

---

## Uso de Inventarios de Ejemplo

El repositorio incluye:

```text
inventory.ini.example
```

Este archivo funciona como plantilla para configurar la infraestructura sin exponer información sensible.

El inventario real:

```text
inventory.ini
```

debe permanecer excluido mediante `.gitignore`.

---

# Idempotencia

Uno de los principios fundamentales de Ansible es la **idempotencia**.

Esto significa que las tareas pueden ejecutarse varias veces y Ansible solamente realizará cambios cuando sean necesarios.

Por ejemplo:

```yaml
- name: Instalar Nginx
  apt:
    name: nginx
    state: present
```

Si Nginx ya está instalado, Ansible no volverá a instalarlo.

De esta forma se consigue una configuración predecible, repetible y consistente.

---

# Despliegue de la Aplicación

El rol `app` utiliza el módulo `unarchive` para desplegar un archivo comprimido:

```text
website.tar.gz
```

El contenido se extrae dentro de:

```text
/var/www/html
```

La aplicación utiliza los permisos correspondientes para que Nginx pueda servir correctamente los archivos.

Estructura esperada:

```text
/var/www/html
├── index.html
├── css/
├── js/
└── assets/
```

---

# Gestión de Nginx

El rol `nginx` se encarga de:

1. Instalar Nginx.
2. Iniciar el servicio.
3. Habilitar el servicio para que arranque automáticamente.
4. Mantener el estado deseado del servidor web.

Estado esperado:

```text
Nginx
├── Instalado
├── Ejecutándose
└── Habilitado al iniciar el sistema
```

---

# Gestión de Seguridad

El rol `base` instala herramientas necesarias para la administración y seguridad básica del servidor:

```text
curl
htop
vim
git
fail2ban
```

`fail2ban` ayuda a proteger servicios expuestos contra intentos repetidos de autenticación, especialmente ataques de fuerza bruta sobre SSH.

---

# Flujo de Automatización

```text
                 Nodo de Control
                  Fedora / Linux
                        |
                        v
                     Ansible
                        |
                        v
                 inventory.ini
                        |
                        v
                 Servidor Ubuntu
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
        base           ssh          nginx
          |             |             |
          +-------------+-------------+
                        |
                        v
                       app
                        |
                        v
                 /var/www/html
                        |
                        v
                  Aplicación Web
```

---

# Tecnologías Utilizadas

| Tecnología | Uso |
| :--- | :--- |
| Ansible | Automatización y gestión de configuración |
| Ubuntu Server | Sistema operativo del servidor |
| Fedora Linux | Nodo de control |
| Nginx | Servidor web |
| SSH | Comunicación segura con el servidor |
| Systemd | Administración de servicios |
| Fail2ban | Protección contra ataques de fuerza bruta |
| Git | Control de versiones |
| IaC | Infraestructura como Código |

---

# Objetivos del Proyecto

Este proyecto busca demostrar conocimientos prácticos en:

- Infraestructura como Código.
- Automatización de servidores.
- Administración de sistemas Linux.
- Administración remota mediante SSH.
- Ansible.
- Arquitectura basada en Roles.
- Idempotencia.
- Gestión de servicios mediante Systemd.
- Seguridad básica de servidores.
- Despliegue automatizado de aplicaciones.
- Uso de inventarios.
- Uso de Tags.
- Buenas prácticas de Git.
- Gestión segura de secretos.

---

# Comandos Principales

### Comprobar versión de Ansible

```bash
ansible --version
```

### Comprobar conectividad

```bash
ansible -i inventory.ini webservers -m ping
```

### Ejecutar todo el proyecto

```bash
ansible-playbook -i inventory.ini setup.yml
```

### Ejecutar únicamente la aplicación

```bash
ansible-playbook -i inventory.ini setup.yml --tags "app"
```

### Ejecutar Nginx y SSH

```bash
ansible-playbook -i inventory.ini setup.yml --tags "nginx,ssh"
```

### Ejecutar en modo comprobación

```bash
ansible-playbook -i inventory.ini setup.yml --check
```

---

# Resultado Esperado

Después de ejecutar correctamente el playbook, el servidor Ubuntu deberá contar con:

```text
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
```

El objetivo final es disponer de un servidor preparado y configurado automáticamente mediante Ansible, reduciendo la intervención manual y permitiendo repetir el proceso de forma consistente sobre diferentes servidores.

---

# Licencia

Este proyecto puede utilizarse con fines educativos y demostrativos para practicar conceptos de DevOps, Linux, Ansible e Infraestructura como Código.
