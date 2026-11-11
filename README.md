# Proyecto de Automatización y Administración de Servidores
### Redes y Telecomunicaciones — UAA (Laboratorio Ansible, Ubuntu Pro y Landscape)

## 1. Integrantes del Equipo
* **Estudiante 1:** Integración técnica, repositorio y nodo de control (`control-ansible`)
* **Estudiante 2:** Servidor Web Apache, rol base e idempotencia (`srv-web-01`)
* **Estudiante 3:** Base de Datos MariaDB, actualización e inventario técnico (`srv-db-01`)
* **Estudiante 4:** Coordinación, VM administrativa, Ubuntu Pro y Landscape (`srv-admin-01`)

## 2. Ficha Técnica del Nodo de Control (`control-ansible`)
* **Sistema Operativo:** Ubuntu 26.04.1 LTS (x86_64)
* **Versión de Ansible:** ansible [core 2.20.1]
* **Intérprete de Python:** /usr/bin/python3 (Python 3.14.4)
* **Versión de Git:** git version 2.53.0
* **Fecha de Configuración:** 2026-10-13

## 3. Inventario y acceso

`control-ansible` administra por SSH a `srv-web-01`, `srv-db-01` y `srv-admin-01`. Cada VM corre en la laptop de su propietario y se conecta mediante Hamachi. Las IP predeterminadas están en [el inventario de laboratorio](inventories/laboratorio/hosts.yml). Para la prueba Hamachi, deja sin definir `IP_WEB`, `IP_DB` e `IP_ADMIN`. `inventories/produccion/` está reservado por separado y sólo contiene un marcador.

Desde la raíz del checkout, con Ansible instalado en `control-ansible`:

```bash
ansible-galaxy collection install -r requirements.yml
ansible-inventory --graph
ansible all -m ansible.builtin.ping
```

SSH usa `automatizacion` y la llave local `~/.ssh/ansible_uaa`. Verifica las huellas SSH con cada propietario. Los playbooks que requieren sudo declaran `become` y se ejecutan con `--ask-become-pass` cuando sudo pide contraseña. La consulta habitual de Pro, el inventario técnico y el ping se ejecutan sin sudo ni contraseña de Vault.

## 4. Integración y demostración

- [A14 y A15 · validación de reportes e integración desde snapshots](docs/integracion.md).
- [A16 · guion y checklist de demostración](docs/demostracion.md).
- [Matriz de evidencias por componente](docs/evidencias/MATRIZ_FINAL_EVIDENCIAS.md).
- [A17 · ficha del baseline y cierre](docs/baseline.md).
- [Problemas conocidos y recuperación](docs/PROBLEMAS_CONOCIDOS.md).

Los reportes se generan en `logs/` del nodo de control, ignorado por Git. Cada propietario revisa su evidencia y conserva bajo `docs/evidencias/` sólo extractos sanitizados. El Vault de Ubuntu Pro permanece cifrado en `vars/vault/vault.yml`. El flujo habitual de registro de Pro y Landscape es interactivo. Los snapshots se gestionan en el hipervisor y se conservan fuera del repositorio.
