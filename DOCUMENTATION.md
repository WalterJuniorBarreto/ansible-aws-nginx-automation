# Conceptos Técnicos de Ansible

Este proyecto demuestra los siguientes principios de la gestión de configuración (Configuration Management):

## 1. Idempotencia
Es la capacidad de ejecutar el mismo script múltiples veces obteniendo siempre el mismo estado final deseado, sin efectos secundarios negativos. Si Ansible detecta que `nginx` ya está instalado o un archivo ya fue copiado, omite la tarea (`changed=0`), ahorrando tiempo y evitando corrupción de datos.

## 2. Arquitectura sin Agentes (Agentless)
A diferencia de Chef o Puppet, Ansible no requiere instalar software esclavo en el servidor destino. Utiliza conexiones estándar SSH y Python para ejecutar los módulos, dejando el servidor limpio tras finalizar.

## 3. Modularidad con Roles
La división en roles (`base`, `nginx`, `app`, `ssh`) permite separar la lógica. Si en el futuro se necesita aprovisionar un servidor de base de datos, se pueden reutilizar los roles `base` y `ssh` sin afectar o instalar dependencias de servidores web.

## 4. Módulos Utilizados
- `apt`: Gestión de paquetes y actualizaciones.
- `service`: Control de demonios de systemd (start, enable).
- `authorized_key`: Inyección segura de claves SSH sin usar `echo` o manipular manualmente `~/.ssh/authorized_keys`.
- `unarchive`: Transferencia y extracción directa de archivos comprimidos desde el nodo de control al nodo destino.

