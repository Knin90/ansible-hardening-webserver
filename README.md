#  Proyecto 1: Servidor Web Reproducible y Hardening Base con Ansible

Este proyecto automatiza la gestión de infraestructura utilizando **Ansible** para aprovisionar un servidor web **Nginx** de manera idempotente sobre Ubuntu Server en AWS EC2, aplicando mejores prácticas de **Hardening SSH** y **Seguridad de Red (UFW)**.

---

##  Objetivos del Proyecto

* **Infraestructura como Código (IaC):** Despliegue reproducible mediante playbooks de Ansible.
* **Hardening SSH:** Restricción del acceso root y creación de un usuario administrador con autenticación por clave pública SSH.
* **Generación Dinámica de Contenido:** Uso de plantillas **Jinja2** para inyectar *Ansible Facts* (IP, memoria RAM, hostname y versión del SO) en tiempo de ejecución.
* **Idempotencia y Handlers:** Uso de *handlers* para reiniciar servicios únicamente cuando la configuración sufra modificaciones reales.

---

##  Arquitectura y Flujo de Ejecución

```text
   [ Administrador / DevOps ]
               │
               ▼  (ansible-playbook site.yml)
     ┌───────────────────┐
     │  Ansible Control  │
     └─────────┬─────────┘
               │
               ├─► 1. Gather Facts (Lectura de IP, RAM, OS, Hostname)
               ├─► 2. Hardening SSH (Usuario 'sysadmin', clave SSH, PermitRootLogin no)
               ├─► 3. Instalación Nginx & Despliegue de Plantilla Jinja2
               └─► 4. Firewall UFW (Apertura estricta de puertos 22/tcp y 80/tcp)
               │
               ▼
     ┌───────────────────┐
     │   Nodo Objetivo   │
     │    (AWS EC2)      │
     └───────────────────┘
```

---

##  Estructura del Proyecto

```text
ansible-hardening-webserver/
├── .gitignore               # Exclusión de archivos sensibles de Git
├── README.md                # Documentación del proyecto
├── ansible.cfg              # Configuración local de Ansible y callbacks
├── inventory.ini            # Inventario de nodos objetivo
├── site.yml                 # Playbook principal de ejecución
├── group_vars/
│   └── webservers.yml       # Variables de grupo (puertos, usuario admin, clave SSH)
└── templates/
    └── index.html.j2        # Plantilla Jinja2 con HTML/CSS e inyección de Facts
```

---

##  Requisitos Previos

1. **Ansible Core** instalado en el nodo de control (Fedora / Linux).

2. Instancia **AWS EC2** (Ubuntu Server LTS) activa.

3. Grupo de Seguridad (*Security Group*) en AWS con tráfico permitido en el puerto `22` (SSH) y `80` (HTTP).

4. Un par de claves SSH en tu máquina local (`~/.ssh/id_ed25519.pub`).

---

## ⚙️ Configuración y Despliegue

### 1. Clonar el Repositorio

```bash
git clone https://github.com/tu_usuario/ansible-hardening-webserver.git
cd ansible-hardening-webserver
```

### 2. Configurar el Inventario (`inventory.ini`)

Actualiza la IP pública de tu servidor y la ruta a tu llave privada `.pem`:

```ini
[webservers]
nodo1 ansible_host=3.17.131.34 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/tu-llave-aws.pem

[webservers:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 3. Configurar las Variables de Grupo (`group_vars/webservers.yml`)

Define el usuario de administración e inyecta tu clave pública SSH local (`cat ~/.ssh/id_ed25519.pub`):

```yaml
---
# Configuración de Usuarios y Hardening SSH
admin_user: sysadmin
ssh_public_key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... tu_email@local"

# Configuración del Servicio Web y Firewall
http_port: 80
ssh_port: 22
```

---

##  Ejecución y Verificación

### Validación de Sintaxis

```bash
ansible-playbook site.yml --syntax-check
```

### Ejecución del Despliegue

```bash
ansible-playbook site.yml
```

### Validación de Idempotencia

Al reejecutar el playbook, la columna `changed` debe mostrar `0`:

```bash
ansible-playbook site.yml
```

**Salida Esperada en Terminal:**

```text
PLAY RECAP **************************************************************************************
nodo1                      : ok=10   changed=0    unreachable=0    failed=0    skipped=0    rescued=0
```

---

##  Consideraciones de Seguridad

* **Acceso Root:** Se deshabilita la directiva `PermitRootLogin` en `/etc/ssh/sshd_config`.

* **Archivos Sensibles:** Las claves privadas (`*.pem`, `id_ed25519`) se encuentran agregadas a `.gitignore` para prevenir filtraciones en repositorios públicos.

---

##  Licencia

Este proyecto está bajo la Licencia **MIT**.

