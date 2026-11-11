# Baseline A17

E4 consolida esta ficha después de aprobar la [demostración A16](demostracion.md). E1 registra el commit demostrado y, tras la aprobación de E4, lo identifica con un tag estable acordado. Si la ficha se actualiza en un commit posterior, conserva la referencia al commit que realmente se probó.

- **Commit y tag estables:** EVIDENCIA PENDIENTE DE EJECUCIÓN.
- **Fecha, responsables y aprobación de E4:** EVIDENCIA PENDIENTE DE EJECUCIÓN.
- **Rama remota auditada antes de los ajustes:** `4d5d8f1` en `main`.
- **Evidencias disponibles:** [matriz por componente](evidencias/MATRIZ_FINAL_EVIDENCIAS.md). Distingue las corridas históricas de las realizadas sobre el commit demostrado.
- **Problemas y mitigaciones:** [registro existente](PROBLEMAS_CONOCIDOS.md).

## Estado de las VM y los servicios

Las direcciones de servidores son los valores predeterminados del [inventario de laboratorio](../inventories/laboratorio/hosts.yml). Los servicios de la tabla identifican qué comprobar al cerrar el baseline.

| VM | Dueño | IP Hamachi registrada | Comprobaciones de estado y versiones |
|---|---|---|---|
| `control-ansible` | E1 | Fuera del inventario administrado. EVIDENCIA PENDIENTE DE EJECUCIÓN. | Ubuntu, kernel, Hamachi, Git, Python, Ansible, colección MariaDB y acceso SSH a los tres nodos. |
| `srv-web-01` | E2 | `25.32.150.213` | Ubuntu, kernel, SSH, Apache activo y HTTP 200. |
| `srv-db-01` | E3 | `25.1.68.192` | Ubuntu, kernel, SSH, versión y escucha local de MariaDB, consulta por socket y reinicio pendiente. |
| `srv-admin-01` | E4 | `25.33.22.147` | Ubuntu, kernel, SSH, estado y versión de Pro, versión y servicio de Landscape, última comunicación en el portal. |

**EVIDENCIA PENDIENTE DE EJECUCIÓN** para estado, versiones vigentes, MTU y snapshots de las cuatro VM. Registra por VM el nombre real, fecha, estado de la máquina al capturar y último ensayo de restauración. Si un punto no existe, déjalo explícito en el registro.

Para confirmar las versiones de Ubuntu, kernel, Hamachi y OpenSSH en cada VM:

```bash
grep '^PRETTY_NAME=' /etc/os-release
uname -r
hamachi --version
ssh -V
```

E1 confirma además `ansible --version`, `python3 --version`, `git --version` y `ansible-galaxy collection list community.mysql`. E2 y E3 usan las comprobaciones de [integración](integracion.md#comandos-de-los-propietarios). E4 ejecuta `pro version` y `dpkg-query -W landscape-client` en la VM administrativa.

## Pendientes de cierre

- [ ] E3 entrega el inventario y los reportes A14 de los tres hosts, más el fallo controlado. E4 revisa la decisión de reinicio.
- [ ] E1 conserva las dos pasadas de A15 y el commit candidato. Los propietarios confirman sus servicios y puntos de restauración.
- [ ] E4 valida la última comunicación de Landscape y el ensayo de A16.
- [ ] E1 y E4 registran el tag estable y completan esta ficha con las evidencias sanitizadas.
