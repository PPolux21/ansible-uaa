# Cierre de A14 e integración A15 desde snapshots

E1 ejecuta la integración en `control-ansible`. E3 valida A14 con cada propietario. E4 coordina la ventana, verifica la recuperación, revisa la [matriz de evidencias](evidencias/MATRIZ_FINAL_EVIDENCIAS.md) y aprueba el cierre. Cada propietario conserva su evidencia.

## Preparación y restauración

- [ ] E1 registra el commit candidato y el estado limpio con `git rev-parse HEAD` y `git status --short`. E4 confirma la ventana, las cuatro laptops, espacio para APT y una decisión de reinicio con los propietarios.
- [ ] Cada propietario registra el nombre real, fecha y estado de los snapshots disponibles. Los nombres sugeridos en el plan son `01-ssh-llaves`, `02-ansible-inventario-validado`, `03-servicios-validados` y `04-gestion-validada`. Conserva los existentes y confirma el punto de retorno previo a A15. Para MariaDB, captura la VM apagada o en estado consistente.
- [ ] Elige con E4 el punto que cada VM restaurará y deja constancia del motivo. Para probar el despliegue de roles, usa el snapshot con SSH e inventario operativos, previo a servicios, si está disponible. Si falta, registra desde qué estado se ensayó. Conserva fuera de la VM el commit y las evidencias que deban sobrevivir a la restauración.
- [ ] Tras restaurar, comprueba hostname, IP de Hamachi, MTU observado, servicio SSH y huella de host con los dueños. E1 recupera el commit candidato en el controlador. E4 verifica si el snapshot de administración conserva los registros de Pro y Landscape.
- [ ] E1 verifica las huellas con los propietarios antes de aceptar una huella nueva o cambiada. Usa la cuenta `automatizacion` y la llave del laboratorio. Consulta [problemas conocidos](PROBLEMAS_CONOCIDOS.md) si falla el enlace.

## Preflight en control-ansible

Ejecuta desde la raíz del repo en Bash. Revisa la salida y detén el ensayo si falta un nodo o falla algún comando.

```bash
unset IP_WEB IP_DB IP_ADMIN
ansible --version
ansible-galaxy collection install -r requirements.yml
ansible-galaxy collection list community.mysql

(
  set -e
  for p in playbooks/*.yml roles/*/tests/test.yml; do
    ansible-playbook --syntax-check "$p"
  done
  ansible-inventory --graph
  ansible all -m ansible.builtin.ping
)
```

Los tres nodos deben aparecer en sus grupos `web`, `databases` y `management` y devolver `pong` por SSH. El grafo y el ping se ejecutan sin sudo ni Vault. Para los playbooks privilegiados, `-K` equivale a `--ask-become-pass` y pide la contraseña sudo de `automatizacion`. Si las contraseñas difieren, ejecuta por host con `--limit` o usa variables `ansible_become_password` por host protegidas con Vault y cargadas explícitamente para esa ejecución. Tres ejecuciones separadas producen tres resúmenes parciales. El cierre de A14 requiere además una corrida normal de los tres nodos en un mismo resumen.

## A14 en control-ansible

E3 ejecuta primero un piloto y revisa sus reportes antes de aplicar la actualización a todos. El fallo controlado usa la ruta de registro y termina con salida distinta de cero. Anota la ruta que imprime cada ejecución.

```bash
ansible-playbook playbooks/inventario.yml
ansible-playbook playbooks/actualizar_servidores.yml --limit srv-db-01 -K
ansible-playbook playbooks/actualizar_servidores.yml --limit srv-db-01 -e simular_fallo=true
ansible-playbook playbooks/actualizar_servidores.yml -K
```

- [ ] Cada propietario coteja su ficha en `logs/inventario/`: hostname, Ubuntu, vCPU, RAM, disco de raíz e IP de `ham0` frente al destino del inventario. Confirma el estado de Pro donde aplique. El [reporte histórico de E3](evidencias/estudiante-3/reporte-inventario.txt) indica «adjunto» para BD en esa corrida. Comprueba el estado real y registra la discrepancia si persiste.
- [ ] En cada `logs/actualizacion/<ejecución>/`, valida un `<host>.log` por nodo seleccionado y un `resumen.log` con los mismos estados. Comprueba UTC de inicio/fin, listas y conteos antes/después, cambio real de paquetes, reinicio y causa del error si aplica. Los guiones indican campos que no se obtuvieron.
- [ ] El piloto termina en `OK`. En la simulación, el host queda en `FAILED`, la causa indica fallo de prueba y no se ejecuta APT. En la corrida completa, los tres quedan `OK`, con `failed=0`, `unreachable=0` y `rescued=0`. Si se pierde SSH, consulta el estado `UNREACHABLE` en el reporte. Ansible difiere el error para poder escribir los archivos, por lo que ese caso puede aparecer como `failed` e `ignored` en el recap.
- [ ] E4 revisa los logs sanitizados y el marcador de reinicio. Acuerda con cada propietario el reinicio si está indicado. `apt` puede dejar paquetes pendientes por las políticas del sistema. Revisa esas diferencias antes de aprobar.
- [ ] Guarda los reportes de A14 con su commit y fecha. **EVIDENCIA PENDIENTE DE EJECUCIÓN** para la revisión actual.

## A15 en control-ansible

Ejecuta la siguiente secuencia en orden. Conserva ambos juegos de recaps y reportes. Si un paso falla, detén la secuencia, corrige la causa y registra la nueva ejecución.
Si es necesario restaurar, cada propietario confirma que APT y los servicios terminaron sus escrituras y vuelve al punto acordado con E4. Conserva los reportes de la corrida fallida.

```bash
ansible-playbook playbooks/preparar_laboratorio.yml -K
ansible-playbook playbooks/aplicar_base.yml -K
ansible-playbook playbooks/deploy_web.yml -K
ansible-playbook playbooks/deploy_mariadb.yml -K
ansible-playbook playbooks/ubuntu_pro.yml
ansible-playbook playbooks/landscape.yml -K
ansible-playbook playbooks/inventario.yml
ansible-playbook playbooks/actualizar_servidores.yml -K
```

Si el snapshot administrativo carece de registro, E4 completa el attach interactivo y el registro Landscape durante la preparación de la primera pasada. Verifica ambos al ejecutar sus respectivos pasos. Con el estado inicial ya registrado, repite la secuencia completa sobre las mismas VM y el mismo commit, sin restaurar otra vez ni hacer cambios manuales entre pasadas. Mantén las dos pasadas dentro de una misma ventana de caché APT si es posible.

- [ ] Ambas pasadas dan `failed=0`, `unreachable=0` y `rescued=0` en la ruta normal. Confirma el número de hosts esperado de cada play.
- [ ] En la segunda pasada, preparación, rol base, Apache, MariaDB, consulta de Pro y Landscape quedan con `changed=0` si se conservan paquetes y servicios. Los cambios de caché por caducidad se explican con la tarea correspondiente. Inventario y actualización generan archivos nuevos en el controlador. Para APT, compara «Actualización cambió paquetes» y las listas de pendientes de ambas pasadas.
- [ ] E2 confirma HTTP 200 y la página esperada con `curl --fail --show-error http://25.32.150.213/` desde el controlador. E3 comprueba la escucha local de MariaDB y ejecuta la consulta por socket. E4 verifica el estado Pro, servicio Landscape y última comunicación en el portal.
- [ ] Tras ambas pasadas, si la política requiere reiniciar, cada dueño lo hace en la ventana acordada y registra el kernel arrancado. E1 repite ping y los responsables vuelven a comprobar sus servicios.
- [ ] E4 aprueba la matriz con vínculos a evidencias de ambas pasadas. E1 etiqueta el commit candidato después de la aprobación. **EVIDENCIA PENDIENTE DE EJECUCIÓN**.

## Comandos de los propietarios

En cada VM, verifica localmente lo que le corresponde. Captura sólo los campos necesarios.

| VM | Comandos y comprobaciones |
|---|---|
| Las cuatro | `hostnamectl --static`, `ip -br -4 addr show dev ham0`, `ip -o link show dev ham0`, `uname -r`. Si hay que comprobar la huella SSH del servidor, ejecuta `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub` en esa VM. |
| Los tres servidores | `systemctl is-active ssh`, `sudo -v` desde la sesión de `automatizacion`, `test ! -f /var/run/reboot-required || cat /var/run/reboot-required`. |
| `control-ansible` | Preflight y secuencia A15 anteriores. |
| `srv-web-01` | `systemctl is-active apache2`, `apache2 -v`, `curl --fail --show-error http://localhost/`. |
| `srv-db-01` | `systemctl is-active mariadb`, `sudo mariadb -N -B -e 'SELECT VERSION();'`, `sudo ss -ltnp 'sport = :3306'`. Verifica escucha en `127.0.0.1:3306`. |
| `srv-admin-01` | `pro status`, `pro security-status`, `systemctl is-enabled landscape-client`, `systemctl is-active landscape-client`. En el portal verifica hostname, versión y última comunicación. |

Si falta el attach de Pro, E4 usa `sudo pro attach` en una consola privada y verifica el resultado. Si falta el registro Landscape, usa `sudo landscape-config` de forma interactiva con los datos del equipo y verifica el portal. Detén grabaciones o capturas durante ambos registros.

Para un attach automatizado expresamente autorizado, usa el Vault existente que contiene `ubuntu_pro_token` y ejecuta `ansible-playbook playbooks/ubuntu_pro.yml --limit srv-admin-01 -K -J -e ubuntu_pro_attach_enabled=true`. El playbook consulta primero si la VM ya está adjunta. Gestiona cualquier cambio de token con `ansible-vault edit vars/vault/vault.yml`. Mantén la contraseña de Vault y los datos de registro fuera de Git.

Antes de guardar evidencias, revisa y redacta tokens, códigos, contraseñas, datos personales, identificadores de suscripción y URLs con credenciales. Conserva sólo los extractos de comandos y reportes que permitan verificar cada criterio.
