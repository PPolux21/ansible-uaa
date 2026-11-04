# Registro de Problemas Conocidos y Soluciones Técnicas Aplicadas

### 1. Congelamiento del Kernel Linux en VirtualBox 7.x (Intel 13ª Gen / Ubuntu 24.04)
* **Síntoma:** Al arrancar o restaurar un snapshot, la VM se congelaba en la rutina del benchmark de instrucciones vectoriales AVX2 (`raid6: avx2x2 gen()`).
* **Causa Raíz:** Incompatibilidad de temporizadores entre el motor de virtualización de VirtualBox sobre Windows 11 con Hyper-V y los snapshots tomados con la máquina encendida (*live state*).
* **Solución:** Configurar la interfaz de paravirtualización en modo **`KVM`**, controlador gráfico **`VMSVGA`** sin aceleración 3D, y tomar snapshots obligatoriamente con la máquina en estado **Apagada** (*en frío*).

### 2. Bloqueo de Ejecución por Cifrado de Ansible Vault
* **Síntoma:** Error `[ERROR]: Attempting to decrypt but no vault secrets found` al ejecutar tareas Ad-Hoc o playbooks.
* **Causa Raíz:** Existencia de archivos cifrados en `group_vars/` (para credenciales de MariaDB o tokens de Ubuntu Pro). Ansible intenta descifrar automáticamente todas las variables al iniciar.
* **Solución:** Incluir la bandera `--ask-vault-pass` (o `-J`) para proporcionar la contraseña maestra en tiempo de ejecución.

### 3. Fallo de Verificación de Huella SSH al Cambiar de Entorno (`Host key verification failed`)
* **Síntoma:** SSH bloqueaba la conexión no interactiva de Ansible al alternar entre la red remota (Hamachi `25.x.x.x`) y la red local del laboratorio (`10.13.x.x`).
* **Causa Raíz:** Las direcciones IP locales no se encontraban registradas en el archivo `~/.ssh/known_hosts`.
* **Solución:** Registrar las huellas criptográficas mediante escaneo previo: `ssh-keyscan -t ed25519 <IP> >> ~/.ssh/known_hosts`.

### 4. Advertencia de Inventario Vacío por Directorio de Ejecución
* **Síntoma:** `[WARNING]: No inventory was parsed, only implicit localhost is available` al ejecutar dentro de `playbooks/`.
* **Causa Raíz:** Ansible busca `./ansible.cfg` en el directorio de trabajo actual. Al estar dentro de una subcarpeta, no encontraba el archivo de configuración de la raíz.
* **Solución:** Ubicarse siempre en la raíz del proyecto (`~/ansible-uaa`) o definir la variable global `export ANSIBLE_CONFIG="$HOME/ansible-uaa/ansible.cfg"` en `~/.bashrc`.
