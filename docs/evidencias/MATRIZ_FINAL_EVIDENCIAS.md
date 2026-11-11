# Matriz de evidencias de integración

La evidencia histórica muestra pruebas previas por componente. E4 completa el cierre cuando puede vincular los artefactos del mismo commit, restauración y ventana de [integración](../integracion.md).

| Componente | Responsable | Evidencia histórica disponible | Evidencia requerida para cierre |
|---|---|---|---|
| Sintaxis, inventario y SSH | E1 | [Grafo](estudiante-1/inventory-graph.txt), [ping](estudiante-1/ping-all.txt) | Syntax-check de todos los playbooks y `pong` de los tres nodos por Hamachi. |
| Preparación y rol base | E2 | [Dos corridas del primer playbook](estudiante-1/tabla_idempotencia.txt), [reporte base](estudiante-2/2026-10-22_reporte_base.txt) | Recaps de ambas pasadas y estado de SSH. |
| Apache | E2 | [Evidencia HTTP](estudiante-2/2026-10-XX_srv-web-01_http.txt), [recap](estudiante-2/deploy-web_2_2.txt) | HTTP 200 desde el controlador y recaps de ambas pasadas. |
| MariaDB | E3 | [Consulta local](estudiante-3/consulta-local-mariadb.txt), [escucha local](estudiante-3/socket-escucha-mariadb.txt), [recap](estudiante-3/run-02-mariadb.txt) | Consulta por socket, escucha sólo local y recaps de ambas pasadas. |
| Ubuntu Pro | E4 | [Estado](estudiante-4/pro-status-despues.txt), [seguridad](estudiante-4/security-status-despues.txt) | Estado adjunto en el snapshot ensayado, salida sanitizada y recaps. |
| Landscape | E4 | [Recap de instalación y servicio](estudiante-4/srv-admin-01_landscape-playbook.txt) | Servicio habilitado y activo, última comunicación del cliente en el portal y recaps. |
| Inventario y actualización A14 | E3 | [Resumen de pruebas](estudiante-3/resumen-actividad-14.txt), [inventario parcial](estudiante-3/reporte-inventario.txt), [actualización OK](estudiante-3/resumen-ok-actualizacion.txt), [fallo controlado](estudiante-3/resumen-failed-actualizacion.txt) | Fichas y logs para los tres servidores, resumen consolidado y fallo controlado de la versión actual. |
| Prueba integral A15 | E1 y E4 | Recaps parciales por componente. [Ejemplo histórico de fallo de enlace](prueba_formato.txt). | Dos pasadas integrales desde la restauración acordada, `failed=0`, `unreachable=0` y `rescued=0`. Verificar `changed=0` donde corresponda. |
| Seguridad y recuperación | E4 y propietarios | [Registro de problemas y mitigaciones](../PROBLEMAS_CONOCIDOS.md) | Confirmación de snapshots, huellas SSH y sanitización de los artefactos nuevos. |

**EVIDENCIA PENDIENTE DE EJECUCIÓN** para los criterios de la última columna.
