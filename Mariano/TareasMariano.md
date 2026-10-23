# Tareas de Mariano, Proyecto 2 BanTech GT

Asignación original y calendario de planificación. El estado actualizado está en el [manual general](../README.md).

Rol: coordinador, backbone y Data Center con Sede Central.

Tu trabajo es el núcleo de la red y el segmento de rutas estáticas. Como Luis es quien une todo en el .pkt maestro, tu entrega tiene que llegarle con la redistribución ya configurada, de modo que al conectar las sedes la red converja sin que él tenga que tocar el backbone.

## Lo que le entregas a Luis

| Entrega | Fecha límite | Contenido |
|---|---|---|
| Plan de direcciones del backbone | Jueves 15 en la noche | Direcciones /30 de todos los enlaces punto a punto, incluyendo los enlaces hacia R-Occidente, R-Norte, MS1, MS2 y R-Central1, para que Oscar y Luis configuren su lado. |
| .pkt de backbone y Central | Domingo 18 a mediodía | Núcleo, R-Central1, R-Central2, switches y servidores del Data Center, con OSPF, EIGRP, RIPv2, estáticas y redistribución ya configurados. |
| Extractos de configuración | Domingo 18 a mediodía | Un .txt por equipo en configs, con lo más importante de cada uno. |
| Sección del manual | Lunes 19 a mediodía | Archivo seccion.md con la justificación del backbone y de Central, el rol de cada protocolo y las capturas de show ip route. |

## Calendario

| Día | Tarea | Duración | Detalle |
|---|---|---|---|
| Mié 14 | Reunión de arranque | 1 h | Con Oscar y Luis. Definir XX, Y, dominio de cada sede, bloques IP por sede y versión de Packet Tracer. |
| Jue 15 | Repositorio privado | 0.5 h | Crear el repositorio con la estructura de carpetas acordada e invitar a Oscar y Luis. |
| Jue 15 | Diseño del backbone | 2 h | Cantidad de equipos del núcleo, forma de interconexión, ubicación de los puntos de redistribución y direcciones /30. Enviar el plan a Luis esa misma noche. |
| Vie 16 | Núcleo en Packet Tracer | 3.5 h | Tres equipos de capa 3, EtherChannel en el núcleo por fibra, un enlace Ethernet y uno serial. OSPF en el núcleo y los routers del lado del backbone hacia los dominios EIGRP y RIPv2. |
| Sáb 17 | Data Center y Sede Central | 3.5 h | Switches de distribución y acceso, VLANs Core_BD (1Y+1), Web_Apps (2Y+1) y NOC (3Y+1) con nombre, EtherChannel de al menos dos cables hacia servidores y rutas estáticas en R-Central2. |
| Dom 18 mañana | Redistribución | 2.5 h | R-Central1 propaga las estáticas hacia el backbone y se configuran al menos dos puntos más entre OSPF, EIGRP y RIPv2. Entregar a Luis a mediodía. |
| Dom 18 noche | Sincronización corta | 0.5 h | Con los tres, sobre el maestro que publicó Luis. Revisar convergencia y mandarle a Luis los comandos de cualquier corrección del backbone. |
| Lun 19 | Evidencias de ruteo | 2 h | Capturas de show ip route en los routers frontera tomadas del maestro y redacción de la sección del manual. Entregar a mediodía. |
| Lun 19 | Revisión final contra la rúbrica | 1.5 h | Con los tres. Recorrer la tabla 8.1 requisito por requisito. |

Total aproximado: 17 horas.

## Requisitos que verificas el lunes

| Requisito | Cómo verificarlo |
|---|---|
| Topología conforme al enunciado | Al menos tres equipos de capa 3 en el núcleo, agregación de enlaces en el núcleo, fibra, Ethernet y serial presentes. |
| RIP, OSPF, EIGRP, HSRP, EtherChannel y VTP funcionando | show ip route con rutas O, D, R y S en los routers frontera, show etherchannel summary con Po en SU, show standby en MS1 y show vtp status en Occidente. |
| Repositorio con commits propios de los tres | Revisar el historial y confirmar que cada uno tiene commits desde su cuenta. |

## Reglas de integración

Luis es el único que edita el .pkt maestro y el README.md. Si encuentras un error en el backbone después del domingo, no edites el maestro. Mándale a Luis los comandos exactos y el equipo donde van, y él los aplica. Tus commits van solo dentro de tu carpeta.

```
README.md                    Luis (manual consolidado)
Unido/BanTech_GT_v1.pkt      Luis (maestro final; ubicación de entrega actual)
Mariano/                    Mariano (.pkt parcial, .conf y README.md)
Oscar/                      Oscar (Occidente y Norte)
Luis/                       Luis (Oriente, integración y evidencias)
Images/                     capturas históricas de backbone y Data Center
```

Nombra las capturas con el equipo y el comando, por ejemplo R-Central1_show-ip-route.png, para que Luis las ubique sin preguntar.
