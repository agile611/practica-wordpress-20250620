## Práctica final: Configuración completa de WordPress

Automatización de la instalación y configuración de un entorno completo de WordPress usando **Ansible**, incluyendo base de datos (MariaDB), servidor web (Apache2 + PHP), balanceador de carga (Nginx) y gestión segura de secretos con **Ansible Vault**.

---

## 📁 Estructura del proyecto

```
wordpress-setup/
├── ansible.cfg
├── hosts
├── site.yml
├── group_vars/
│   └── all/
│       └── vault.yml
├── roles/
│   ├── mariadb/
│   ├── php/
│   ├── apache2/
│   ├── wordpress/
│   └── nginx/
```

Cada rol contiene sus propios `tasks` y, cuando corresponde, plantillas Jinja2 para archivos de configuración.

---

## ⚙️ Requisitos previos

- Ansible instalado en tu máquina de control.
- Acceso SSH a los hosts definidos en el inventario.
- Python y pip en los hosts remotos (para módulos como `community.mysql`).
- Claves SSH configuradas para acceso sin contraseña.
- **PyMySQL** instalado en los hosts que gestionan MariaDB (`ansible.builtin.pip` se encarga en el playbook).

---

## 🚦 Inventario de hosts (`hosts`)

Ejemplo de inventario:

```
[database]
192.168.11.20 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa

[web]
192.168.11.40 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa

[loadbalancer]
192.168.11.30 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

## 🔐 Gestión de secretos con Ansible Vault

Los datos sensibles (usuarios y contraseñas) se almacenan cifrados en `group_vars/all/vault.yml`.

### Crear y cifrar el archivo vault

```sh
mkdir -p group_vars/all
nano group_vars/all/vault.yml
```

Ejemplo de contenido:

```yaml
mariadb_root_password: "SuperMariaDBRootPass"
wordpress_db_name: "wordpress"
wordpress_db_user: "wp_user"
wordpress_db_password: "SuperWPPass"
```

Cifrar el archivo:

```sh
ansible-vault encrypt group_vars/all/vault.yml
```

---

## 🚀 Ejecución del playbook

Lanza el despliegue completo con:

```sh
ansible-playbook -i hosts site.yml --ask-vault-pass
```

Se te pedirá la contraseña de Vault para descifrar los secretos.

---

## 📜 Descripción de los roles principales

- **mariadb**: Instala y configura MariaDB, crea la base de datos y el usuario para WordPress.
- **php**: Instala PHP y las extensiones necesarias.
- **apache2**: Instala Apache2, configura el VirtualHost para WordPress y habilita el sitio.
- **wordpress**: Descarga, descomprime y configura WordPress, ajustando permisos y el archivo `wp-config.php`.
- **nginx**: Instala y configura Nginx como balanceador de carga.

---

## 📝 Ejemplo de ejecución de roles (`site.yml`)

```yaml
---
- name: Configuración del servidor de base de datos (MariaDB)
  hosts: database
  roles:
    - mariadb

- name: Configuración del servidor web (PHP, Apache2, WordPress)
  hosts: web
  roles:
    - php
    - apache2
    - wordpress

- name: Configuración del balanceador de carga (Nginx)
  hosts: loadbalancer
  roles:
    - nginx
```

---

## ✅ Validación

Accede desde tu navegador a [http://192.168.11.30](http://192.168.11.30) (IP del balanceador).  
Deberías ver el instalador de WordPress listo para completar la instalación web.

---

## 🛠️ Notas adicionales

- Puedes modificar los valores de los secretos en `vault.yml` antes de cifrarlo.
- Si añades más servidores, solo actualiza el inventario y ejecuta el playbook.
- Para modificar la configuración de Apache, Nginx o WordPress, edita las plantillas Jinja2 en cada rol.

---

## 📚 Referencias

- [Documentación oficial de Ansible](https://docs.ansible.com/)
- [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
- [WordPress](https://wordpress.org/)

---