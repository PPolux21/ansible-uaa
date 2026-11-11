# Problemas conocidos y mitigaciones

## Hamachi, MTU y SSH

Los nodos están en cuatro laptops. La prueba integral depende de que las cuatro VM y Hamachi estén disponibles. Antes de empezar, confirma las direcciones de `ham0` y compara con el inventario de laboratorio.

En las pruebas del equipo, el intercambio SSH fallaba con MTU 1500 y 1400 y funcionó con 1300 o menos. Inspecciona el MTU de `ham0` con `ip -o link show dev ham0` tras restaurar. Si el síntoma reaparece, verifica la ruta con los propietarios y recupera el ajuste probado donde sea necesario. Registra el valor por VM.

Si cambia la huella SSH al restaurar, contrástala con la huella de la VM obtenida por el propietario con `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`. Actualiza sólo la entrada comprobada de `known_hosts` y mantén habilitada la verificación.

## Sudo y Vault

El usuario SSH es `automatizacion`. Para tareas que necesitan root, Ansible usa sudo y, si se requiere, `-K` solicita la contraseña de ese usuario. Si falla, prueba `sudo -v` como `automatizacion` en el nodo afectado y revisa la política sudo. La cuenta root puede seguir bloqueada para login directo.

El Vault cargado automáticamente desde `group_vars/management` era la causa histórica de los errores de descifrado en operaciones simples. El cifrado se conserva en `vars/vault/vault.yml`. El play de Ubuntu Pro lo abre sólo cuando se autoriza un attach y la VM aún está sin adjuntar. Ping y las consultas habituales funcionan sin `-J`.

## APT, kernel y reinicio

El desempaquetado de kernel y headers de `srv-admin-01` tardó considerablemente y luego continuó en APT. Si vuelve a ocurrir, comprueba el avance de `apt-get` y `dpkg` en la VM y el registro local de dpkg. Espera mientras el trabajo progrese. Una espera larga por sí sola no demuestra un fallo.

El playbook registra `/var/run/reboot-required` y los paquetes asociados si están disponibles. E4 y el propietario deciden cuándo reiniciar. Al recuperar la VM, comprueba `uname -r`, ping y los servicios del perfil.

## Arranque y directorio de trabajo

En el caso previo de VirtualBox 7.x sobre Windows 11 con Hyper-V, el equipo registró una congelación al restaurar un snapshot activo durante el arranque del kernel. Para la VM afectada, la mitigación documentada fue usar paravirtualización KVM, VMSVGA sin aceleración 3D y snapshots con la VM apagada. Verifica la configuración del hipervisor con su propietario si reaparece.

Ejecuta Ansible desde la raíz del repositorio para que cargue `ansible.cfg`, el inventario y los roles esperados.
