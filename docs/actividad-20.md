# Actividad 20: Respaldo y recuperación de MariaDB

Responsable: Estudiante 3 (`srv-db-01`, grupo `databases`).

## Objetivo

Demostrar que los datos de MariaDB pueden respaldarse y recuperarse de forma controlada: se crea una base de prueba, se respalda, se provoca una pérdida controlada, se restaura y se verifica que los datos recuperados son idénticos a los respaldados.

## Archivos

| Archivo | Función |
|---|---|
| `playbooks/respaldo_mariadb.yml` | Playbook con todas las fases del respaldo y la recuperación |
| `templates/respaldo_reporte.log.j2` | Plantilla del reporte generado en el nodo de control |
| `docs/evidencias/estudiante-3/actividad-20/` | Evidencias de la ejecución completa |

## Qué hace el playbook

El playbook se ejecuta sobre el grupo `databases` con `become: true`. Todas las variables llevan el prefijo `resp_` y están definidas en el play. Las tareas están agrupadas en fases con etiquetas:

| Etiqueta | Acción |
|---|---|
| `preparar` | Crea la base `prueba_respaldo`, la tabla `registros` (id, texto, fecha; InnoDB) y 8 registros con valores fijos. Solo crea lo que falta. |
| `consulta_antes` | Muestra el conteo de filas y el `CHECKSUM TABLE` de la tabla. |
| `respaldar` | Verifica que el directorio de respaldos está fuera del directorio de datos activo, crea `/var/backups/mariadb/`, genera el respaldo comprimido y un archivo de metadatos. |
| `perdida` | Comprueba que la base es `prueba_respaldo` y que existe al menos un respaldo; después borra la tabla y confirma que ya no existe. |
| `restaurar` | Localiza el respaldo más reciente, comprueba su SHA-256 contra los metadatos e importa el respaldo en `prueba_respaldo`. |
| `verificar` | Compara el conteo y el checksum actuales contra los metadatos del respaldo, revisa que MariaDB escucha solo en `127.0.0.1`, genera el reporte y termina con resultado `OK` o `FAILED`. |

Como las fases pueden ejecutarse por separado, la comparación de `verificar` toma los valores de "antes" del archivo de metadatos del respaldo, no de variables de la misma ejecución.

## Comandos

Desde la raíz del repositorio en el nodo de control (`control-ansible`). El inventario por defecto es `inventories/laboratorio/hosts.yml` (definido en `ansible.cfg`), y `become_ask_pass = True` solicita la contraseña de sudo.

Ejecución completa:

```bash
ansible-playbook playbooks/respaldo_mariadb.yml
```

Ejecución por fases:

```bash
ansible-playbook playbooks/respaldo_mariadb.yml --tags preparar,consulta_antes
ansible-playbook playbooks/respaldo_mariadb.yml --tags respaldar
ansible-playbook playbooks/respaldo_mariadb.yml --tags perdida
ansible-playbook playbooks/respaldo_mariadb.yml --tags restaurar,verificar
```

Comprobación manual en `srv-db-01` de que la tabla desaparece y vuelve:

```bash
sudo mariadb -e 'SHOW TABLES FROM prueba_respaldo; SELECT * FROM prueba_respaldo.registros;'
sudo mariadb -e 'CHECKSUM TABLE prueba_respaldo.registros;'
```

Después de `perdida` la consulta devuelve `ERROR 1146 ... Table 'prueba_respaldo.registros' doesn't exist`; después de `restaurar` aparecen las 8 filas con el mismo checksum.

## Ubicación de los archivos generados

| Elemento | Ubicación | Permisos |
|---|---|---|
| Directorio de respaldos | `srv-db-01:/var/backups/mariadb/` | `root:root` `0700` |
| Respaldo | `/var/backups/mariadb/prueba_respaldo_<AAAA-MM-DD_HHMMSS>.sql.gz` | `root:root` `0600` |
| Metadatos | `/var/backups/mariadb/prueba_respaldo_<AAAA-MM-DD_HHMMSS>.meta.json` | `root:root` `0600` |
| Reporte | nodo de control: `logs/respaldo/<AAAA-MM-DD_HHMMSS>/reporte.log` | `0644` |

El archivo de metadatos (JSON) registra servidor, base, tabla, fecha, herramienta, directorio de datos, ruta del respaldo, tamaño en bytes, SHA-256, conteo de filas y checksum de la tabla al momento del respaldo.

El reporte registra servidor, id y fecha de la ejecución, archivo de respaldo, fecha, tamaño, SHA-256 e integridad del respaldo, conteo y checksum antes y después, direcciones de escucha del puerto 3306 y el resultado de la recuperación. El directorio `logs/` está excluido del repositorio en `.gitignore`.

El id de ejecución es la fecha y hora del servidor al recolectar facts; coincide con el nombre del respaldo generado en la misma ejecución.

## Resultado de la prueba

Ejecución completa del 2026-11-12 en el entorno de pruebas (`reporte-respaldo.txt`):

| Dato | Valor |
|---|---|
| Respaldo | `/var/backups/mariadb/prueba_respaldo_2026-11-12_171646.sql.gz` |
| Tamaño | 1008 bytes |
| SHA-256 | `e1bf1ae04fb839787bf171ae8f86adc0d2167d7f401f0c3ec5c98efd51552ab7` |
| Antes | 8 filas, `CHECKSUM TABLE` 2321021612 |
| Después | 8 filas, `CHECKSUM TABLE` 2321021612 |
| Puerto 3306 | solo `127.0.0.1:3306` |
| Resultado | `OK` |
| PLAY RECAP | `ok=39 changed=7 failed=0 skipped=2` |

Evidencias en `docs/evidencias/estudiante-3/actividad-20/`:

| Archivo | Contenido |
|---|---|
| `run-01-respaldo.txt` | Salida de la ejecución completa del playbook |
| `reporte-respaldo.txt` | Reporte generado en el nodo de control |
| `revision-vm-respaldo.txt` | Permisos del directorio y archivos, SHA-256 del respaldo y búsqueda de contraseñas en el respaldo (0 coincidencias) |

## Justificación de las decisiones

### Herramienta de respaldo

Se usa `community.mysql.mysql_db` con `state: dump` y `state: import`. Con MariaDB 10.4.6 o superior el módulo invoca `mariadb-dump` y `mariadb` (en el servidor, `mysqldump` es solo un enlace a `mariadb-dump`). Un respaldo lógico es adecuado para una base pequeña: produce SQL legible y portable.

- `single_transaction: true` agrega `--single-transaction`, que obtiene una vista consistente de las tablas InnoDB sin bloquearlas.
- La extensión `.sql.gz` hace que el módulo comprima con `gzip` y use `pipefail`, de modo que un fallo de `mariadb-dump` no queda oculto por la compresión.
- En la importación el módulo ejecuta `mariadb --one-database prueba_respaldo`, que ignora sentencias dirigidas a otras bases.

El módulo evita escribir el comando con `shell` y mantiene el mismo mecanismo de conexión que el rol `mariadb`.

### Ubicación y permisos

En MariaDB 11.8 de Ubuntu el directorio de datos activo es `/var/lib/mariadb/`, no `/var/lib/mysql`. El playbook consulta `@@datadir` y un `assert` comprueba que `/var/backups/mariadb` no está dentro de él ni lo contiene. Guardar el respaldo fuera del directorio de datos evita perderlo junto con los datos y que el servidor lo trate como parte de su almacenamiento.

El directorio queda `root:root` `0700` y los archivos `0600`, porque un respaldo contiene todos los datos de la base. El módulo crea el archivo con la máscara de root (`0644`), por eso una tarea posterior fija `0600`; mientras tanto, el directorio `0700` impide que otros usuarios lo lean.

### Credenciales: Vault y `no_log` no aplican

Todas las tareas de MariaDB se conectan como root por `login_unix_socket` (`/run/mysqld/mysqld.sock`). La autenticación `unix_socket` valida que el proceso es root del sistema, por lo que no existe contraseña de base de datos que cifrar con Vault ni que ocultar con `no_log`. Es el mismo criterio aplicado en el rol `mariadb` (Actividad 11). El respaldo tampoco contiene contraseñas, porque solo incluye la base `prueba_respaldo` y no las tablas de usuarios del servidor; la revisión del respaldo da 0 coincidencias de `password`, `identified` o `passwd`.

La única contraseña involucrada es la de sudo, que Ansible solicita en cada ejecución (`become_ask_pass`) y no se guarda en ningún archivo.

### Puerto 3306

El playbook no modifica la configuración de MariaDB. La fase `verificar` consulta los sockets en escucha con `ss -ltnH sport = :3306` (solo lectura) y un `assert` exige que la única dirección sea `127.0.0.1:3306`. Todo el respaldo y la restauración se realizan por socket local, sin necesidad de abrir el puerto a la red.

### Idempotencia

- `preparar` consulta `information_schema` y los ids existentes antes de crear la tabla o insertar registros, por lo que una segunda ejecución reporta `changed=0`. No se usa `CREATE TABLE IF NOT EXISTS` porque `mysql_query` lo reporta como cambio aunque la tabla ya exista.
- Las consultas (`consulta_antes`, conteos, checksum, `@@datadir`, `ss`) usan `changed_when: false`.
- `respaldar`, `perdida` y `restaurar` son acciones: cada ejecución genera un respaldo nuevo, borra la tabla y la importa. Reportan `changed` siempre, y eso es lo esperado. En la ejecución completa, `changed=7` corresponde a: respaldo, permisos del respaldo, metadatos, borrado de la tabla, importación, directorio del reporte y reporte.

El SHA-256 cambia en cada respaldo aunque los datos sean los mismos, porque `mariadb-dump` escribe la fecha al final del archivo y `gzip` guarda una marca de tiempo. Por eso la igualdad de los datos se comprueba con el conteo y el `CHECKSUM TABLE`, y el SHA-256 se usa para verificar la integridad de cada archivo.

### Seguridad del paso destructivo

- Antes de borrar, un `assert` exige `resp_db == 'prueba_respaldo'`; con cualquier otro valor la ejecución se detiene sin borrar nada.
- Un segundo `assert` exige que exista al menos un respaldo de la base en `/var/backups/mariadb/`.
- Solo se borra la tabla de prueba (`DROP TABLE` dentro de `prueba_respaldo`), nunca la base ni otras bases.
- Antes de restaurar se repite el `assert` de la base y se comprueba que el SHA-256 del respaldo coincide con sus metadatos.
- La importación se limita a `prueba_respaldo` (`--one-database`).
- El reporte se genera antes de los `assert` finales, de modo que un resultado `FAILED` también queda registrado.

## Pendientes

- Los respaldos se acumulan en `/var/backups/mariadb/`; el playbook no elimina respaldos antiguos. Para la base de prueba el espacio es mínimo (alrededor de 1.5 KB por ejecución), pero un uso continuo requeriría una política de retención.
