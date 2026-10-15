# Tareas de Luis, Proyecto 2 BanTech GT

Rol: Sede Oriente, integración del .pkt maestro y del manual técnico.

Eres quien une todo. Mariano y Oscar te entregan sus sedes ya configuradas, incluyendo el enrutamiento hacia el backbone, y tú las fusionas en el .pkt maestro, consolidas el README.md y aplicas cualquier corrección que te manden. Eres el único que edita esos dos archivos, para que nunca existan dos versiones distintas del maestro.

## Lo que recibes y cuándo

| De | Entrega | Fecha límite |
|---|---|---|
| Mariano | Plan de direcciones del backbone (/30 de todos los enlaces, incluidos los de MS1 y MS2) | Jueves 15 en la noche |
| Oscar | VLSM de Occidente y Norte | Jueves 15 en la noche |
| Oscar | .pkt de Occidente y Norte con routers de borde configurados, más extractos | Sábado 17 en la noche |
| Mariano | .pkt de backbone y Central con redistribución, más extractos | Domingo 18 a mediodía |
| Mariano y Oscar | seccion.md de cada uno con justificaciones y capturas | Lunes 19 a mediodía |

Si alguna entrega se atrasa, avisa en el grupo de inmediato, porque la fusión del domingo depende de que todo llegue a tiempo.

## Calendario

| Día | Tarea | Duración | Detalle |
|---|---|---|---|
| Mié 14 | Reunión de arranque | 1 h | Con Mariano y Oscar. Definir XX, Y, dominio de cada sede, bloques IP por sede y versión de Packet Tracer. |
| Jue 15 | VLSM de Oriente y Central | 1 h | Bóveda 40 y Plataforma 60 hosts, más Core_BD 14, Web_Apps 28 y NOC 10 para pasárselo a Mariano. |
| Jue 15 | Plantilla de sección | 0.5 h | Subir a la raíz una plantilla de seccion.md con los encabezados de la 4.9, para que Mariano y Oscar escriban en el mismo formato. |
| Jue 15 noche | Tabla maestra | 1.5 h | Consolidar el subneteo y las VLANs de las cuatro sedes y el backbone en el README con lo que te envíen Mariano y Oscar. |
| Vie 16 | Oriente completo | 3 h | MS1 y MS2 en cascada, acceso conectado a ambos, VLANs 8Y y 9Y con nombre y SVIs, HSRP con prioridad y preempt en MS1. |
| Sáb 17 | Prueba de HSRP y dominio de Oriente | 2 h | Apagar la interfaz de MS1 y confirmar que MS2 toma el gateway sin perder pings. Configurar el protocolo de Oriente hacia el backbone con las direcciones del plan. |
| Sáb 17 | Esqueleto del README | 1 h | Secciones de la 4.9 listas para pegar capturas y textos. |
| Dom 18 mañana | Primera fusión | 1.5 h | Copiar Occidente, Norte y Oriente al maestro. |
| Dom 18 tarde | Segunda fusión y convergencia | 2.5 h | Agregar backbone y Central apenas llegue la entrega de Mariano, cablear los routers de borde, verificar tablas de rutas y pings de extremo a extremo. Etiquetar en la topología el área de cada protocolo y revisar hostnames. Publicar el maestro en el repositorio. |
| Dom 18 noche | Sincronización corta | 0.5 h | Con los tres, sobre el maestro publicado. Anotar las correcciones que cada uno te mande. |
| Lun 19 mañana | Correcciones y capturas | 1.5 h | Aplicar las correcciones de la noche anterior, volver a publicar el maestro, tomar las capturas de HSRP y armar la matriz de pings entre sedes. |
| Lun 19 tarde | Unión del manual | 2 h | Integrar las secciones de Mariano y Oscar en el README.md, agregar el mapeo lógico de troncales y puertos de acceso y revisar la redacción completa. |
| Lun 19 | Revisión final contra la rúbrica | 1.5 h | Con los tres. Recorrer la tabla 8.1 y hacer el commit final del maestro y del README. |

Total aproximado: 19.5 horas.

## Requisitos que verificas el lunes

| Requisito | Cómo verificarlo |
|---|---|
| Subnetting VLSM o FLSM correcto | Ninguna red se traslapa, cada VLAN alcanza sus hosts requeridos y las direcciones del .pkt coinciden con la tabla del README. |
| README.md con todo lo de la sección 4.9 | Capturas de la topología, justificación de cada sede y del backbone, extractos de configuración por equipo, tabla de subneteo, tabla de VLANs y capturas de conectividad. |

## Reglas de integración

Solo tú editas el .pkt maestro y el README.md. Cuando Mariano u Oscar encuentren un error después de la fusión, te mandan los comandos exactos y el equipo, tú los aplicas y vuelves a publicar el maestro con un commit que diga qué cambió. Las pruebas de falla de los demás se hacen sobre copias del maestro.

```
README.md                    Luis
BanTech_GT.pkt               Luis
sedes/backbone-central/      Mariano
sedes/occidente-norte/       Oscar
sedes/oriente/               Luis (.pkt parcial, configs, seccion.md)
img/                         una carpeta de capturas por sede
```

Pide que las capturas vengan nombradas con el equipo y el comando, por ejemplo MS1_show-standby.png, así las enlazas en el README sin tener que abrirlas una por una.