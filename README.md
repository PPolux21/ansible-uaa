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
## 3. Estructura de Inventarios
* Laboratorio: `inventories/laboratorio/hosts.yml` (Conectividad sobre red privada Hamachi)
* Producción: `inventories/produccion/hosts.yml` (Separado y protegido)
