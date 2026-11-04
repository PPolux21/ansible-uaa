# Matriz Final de Evidencias de Integración — Proyecto Ansible UAA
| Requerimiento | Servidor Objetivo | Evidencia Comprobada | Estado |
| :--- | :--- | :--- | :---: |
| **Transporte Seguro** | `all` | Ping universal con llaves Ed25519 sobre Hamachi/LAN | APROBADO |
| **Rol Base** | `all` | OpenSSH, DAR y Mutt instalados | APROBADO |
| **Servidor Web** | `srv-web-01` | Apache2 activo, template dinámico y respuesta HTTP 200 | APROBADO |
| **Base de Datos** | `srv-db-01` | MariaDB activo, socket local funcional y puerto 3306 cerrado en red | APROBADO |
| **Ubuntu Pro** | `srv-admin-01` | Conexión ESM activa y sanitizada sin fuga de tokens | APROBADO |
| **Landscape** | `srv-admin-01` | Cliente registrado y monitoreado | APROBADO |
| **Operación y Logs** | `all` | Reportes de facts y actualización generados | APROBADO |
| **Idempotencia** | `all` | Segunda pasada con `changed=0`, `failed=0` y `unreachable=0` | APROBADO |
| **Seguridad Git** | Repositorio | Historial auditado sin contraseñas ni claves privadas | APROBADO |
