# Tareas de Oscar, Proyecto 2 BanTech GT

Rol: Sede Occidente y Sede Norte.

Tus dos sedes son de capa 2 y se pueden construir sin depender del backbone. Como Luis es quien une todo en el .pkt maestro, tu entrega tiene que llegarle con los routers de borde ya configurados con su protocolo de enrutamiento y las direcciones del plan de Mariano, de modo que al conectarlos al núcleo la sede aparezca sola en las tablas de rutas.

## Lo que le entregas a Luis

| Entrega | Fecha límite | Contenido |
|---|---|---|
| VLSM de Occidente y Norte | Jueves 15 en la noche | Red, máscara, gateway y rango de cada VLAN, para la tabla maestra. |
| .pkt de Occidente y Norte | Sábado 17 en la noche | Ambas sedes completas, con R-Occidente y R-Norte ya configurados con su protocolo hacia el backbone. |
| Extractos de configuración | Sábado 17 en la noche | Un .txt por equipo en configs, con lo más importante de cada uno. |
| Sección del manual | Lunes 19 a mediodía | Archivo seccion.md con los párrafos de justificación de cada sede, la elección del Root Bridge y las capturas de la prueba del anillo. |

## Calendario

| Día | Tarea | Duración | Detalle |
|---|---|---|---|
| Mié 14 | Reunión de arranque | 1 h | Con Mariano y Luis. Definir XX, Y, dominio de cada sede, bloques IP por sede y versión de Packet Tracer. |
| Jue 15 | VLSM de Occidente y Norte | 1.5 h | Cajas 45, Asesores 30, Gerencia 10, Seguridad 12, Análisis 50, Auditoría 25 y Legal 14 hosts. Enviar a Luis esa misma noche. |
| Jue 15 | Topología física de Occidente | 1 h | Colocar R-Occidente, los switches de acceso y los 7 equipos mínimos por área. |
| Vie 16 | Occidente completo | 3 h | VLANs 1Y a 4Y con nombre, VTP bantechXX con contraseña cisco, subinterfaces 802.1Q en R-Occidente y prueba de inter-VLAN. |
| Sáb 17 | Norte completo | 3.5 h | Triángulo o anillo de switches, Rapid PVST+, prioridad para forzar el Root Bridge, VLANs 5Y a 7Y con nombre e inter-VLAN. |
| Sáb 17 | Routers de borde | 1 h | Protocolo acordado en R-Occidente y R-Norte, con las direcciones /30 del plan de Mariano. Entregar a Luis esa noche. |
| Dom 18 noche | Sincronización corta | 0.5 h | Con los tres, sobre el maestro que publicó Luis. |
| Dom 18 noche | Verificación de tus sedes en el maestro | 1 h | Pings de Occidente y Norte hacia el Data Center y entre sus VLANs. Mandarle a Luis los comandos de cualquier corrección. |
| Lun 19 | Falla del anillo STP | 2 h | Sobre una copia del maestro, apagar un enlace del anillo de Norte y capturar la reconvergencia. Escribir la sección del manual y entregarla a mediodía. |
| Lun 19 | Revisión final contra la rúbrica | 1.5 h | Con los tres. Recorrer la tabla 8.1 requisito por requisito. |

Total aproximado: 16 horas.

## Requisito que verificas el lunes

| Requisito | Cómo verificarlo |
|---|---|
| Evidencia de pings, inter-VLAN y convergencia | Confirmar que el README tiene capturas de ping entre VLANs de una misma sede y de extremo a extremo (por ejemplo Occidente al Data Center). |

## Reglas de integración

Luis es el único que edita el .pkt maestro y el README.md. Las pruebas de falla del lunes se hacen sobre una copia del maestro, nunca sobre el original. Si encuentras un error en tus sedes después del domingo, mándale a Luis los comandos exactos y el equipo donde van, y él los aplica. Tus commits van solo dentro de tu carpeta.

```
README.md                    Luis
BanTech_GT.pkt               Luis
sedes/backbone-central/      Mariano
sedes/occidente-norte/       Oscar (.pkt parcial, configs, seccion.md)
sedes/oriente/               Luis
img/occidente-norte/         capturas de Oscar
```

Nombra las capturas con el equipo y el comando, por ejemplo SW-N-ROOT_show-spanning-tree.png, para que Luis las ubique sin preguntar.