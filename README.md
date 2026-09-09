# Configuration Management con Ansible

Automatización y gestión de configuración de un servidor Linux remoto utilizando Ansible. Este proyecto aprovisiona un servidor desde cero instalando utilidades base, configurando seguridad, levantando un servidor web y desplegando una aplicación estática.

## Estructura del Proyecto
```text
.
├── inventory.ini    # Archivo de inventario (ignorado en Git por seguridad)
├── setup.yml        # Playbook principal que orquesta los roles
└── roles/           # Módulos reutilizables
    ├── base/        # Actualizaciones, herramientas y fail2ban
    ├── ssh/         # Gestión de llaves públicas autorizadas
    ├── nginx/       # Instalación y configuración del servidor web
    └── app/         # Despliegue del código de la aplicación (.tar.gz)
Guía de Ejecución
1. Definir el inventario
Crea un archivo inventory.ini en la raíz del proyecto:

Ini, TOML
[webservers]
<IP_DEL_SERVIDOR> ansible_user=ubuntu ansible_ssh_private_key_file=/ruta/a/tu/llave.pem
[webservers:vars]
ansible_python_interpreter=/usr/bin/python3
2. Ejecutar aprovisionamiento completo
Bash
ansible-playbook -i inventory.ini setup.yml
3. Ejecutar roles específicos (Tags)
Bash
ansible-playbook -i inventory.ini setup.yml --tags "nginx,app"
