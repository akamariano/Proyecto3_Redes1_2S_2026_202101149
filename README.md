# Proyecto3_Redes1_2S_2026_202101149
# Tareas asignadas, Proyecto 2 BanTech GT

Redes de Computadoras 1, segundo semestre 2026. Grupo de tres integrantes (Mariano, Oscar y Luis).

El arranque es hoy miércoles 14 de octubre en la noche y el trabajo va del jueves 15 al lunes 19. La elaboración oficial cierra el 23 y la calificación es el 24, así que del 20 al 23 queda como margen para corregir errores y preparar la defensa.

## Reparto general

| Integrante | Responsabilidad | Horas propias |
|---|---|---|
| Mariano | Coordinación, backbone y Data Center con Sede Central | 14 h |
| Oscar | Sede Occidente y Sede Norte | 13.5 h |
| Luis | Sede Oriente, direccionamiento consolidado y manual técnico | 12.5 h |
| Los tres | Reunión de arranque, fusión del .pkt maestro, sincronización y revisión final | 5 h |

Mariano lleva el backbone y el Data Center porque ahí se concentran los puntos de redistribución, y alguien necesita ver la red completa para integrarla. Oscar toma Occidente y Norte, que son de capa 2 (VTP, Router-on-a-Stick y Rapid PVST+) y no dependen del núcleo hasta el domingo. Luis toma Oriente con HSRP y además consolida el subneteo y el README, que es lo que más se cruza con las demás sedes.

## Miércoles 14, arranque

| Responsable | Tarea | Duración | Detalle |
|---|---|---|---|
| Los tres | Reunión de arranque | 1 h | Definir XX, Y, dominio de enrutamiento de cada sede, bloques IP por sede, versión de Packet Tracer y quién crea el repositorio. |

## Jueves 15, diseño y subnetting

| Responsable | Tarea | Duración | Detalle |
|---|---|---|---|
| Mariano | Repositorio privado | 0.5 h | Crear el repositorio, la estructura de carpetas e invitar a Oscar y Luis. |
| Mariano | Diseño del backbone | 2 h | Cantidad de equipos del núcleo, forma de interconexión, ubicación de los puntos de redistribución y direcciones /30 de los enlaces punto a punto. |
| Oscar | VLSM de Occidente y Norte | 1.5 h | Cajas 45, Asesores 30, Gerencia 10, Seguridad 12, Análisis 50, Auditoría 25 y Legal 14 hosts. |
| Oscar | Topología física de Occidente | 1 h | Colocar R-Occidente, los switches de acceso y los 7 equipos mínimos por área. |
| Luis | VLSM de Oriente y Central | 1 h | Bóveda 40 y Plataforma 60 hosts, más Core_BD 14, Web_Apps 28 y NOC 10 para pasárselo a Mariano. |
| Luis | Tabla maestra | 1.5 h | Consolidar el subneteo y las VLANs de las cuatro sedes y el backbone en el README. |

## Viernes 16, primeras sedes

| Responsable | Tarea | Duración | Detalle |
|---|---|---|---|
| Mariano | Núcleo en Packet Tracer | 3 h | Tres equipos de capa 3, EtherChannel en el núcleo por fibra, un enlace Ethernet y uno serial. OSPF en el núcleo. |
| Oscar | Occidente completo | 3 h | VLANs 1Y a 4Y con nombre, VTP bantechXX con contraseña cisco, subinterfaces 802.1Q en R-Occidente y prueba de inter-VLAN. |
| Luis | Oriente completo | 3 h | MS1 y MS2 en cascada, acceso conectado a ambos, VLANs 8Y y 9Y con SVIs, HSRP con prioridad y preempt en MS1. |

## Sábado 17, sedes restantes

| Responsable | Tarea | Duración | Detalle |
|---|---|---|---|
| Mariano | Data Center y Sede Central | 3.5 h | Switches de distribución y acceso, VLANs Core_BD, Web_Apps y NOC, EtherChannel de al menos dos cables hacia servidores y rutas estáticas en R-Central2. |
| Oscar | Norte completo | 3.5 h | Triángulo o anillo de switches, Rapid PVST+, prioridad para forzar el Root Bridge, VLANs 5Y a 7Y e inter-VLAN. |
| Luis | Prueba de HSRP y dominio de Oriente | 2 h | Apagar la interfaz de MS1 y confirmar que MS2 toma el gateway sin perder pings. Configurar el protocolo de enrutamiento de Oriente. |
| Luis | Esqueleto del README | 1 h | Secciones de la 4.9 listas para pegar capturas. |

## Domingo 18, integración

| Responsable | Tarea | Duración | Detalle |
|---|---|---|---|
| Los tres | Fusión del .pkt maestro | 2 h | Copiar cada sede al archivo maestro y conectar los routers de borde al backbone, en llamada para resolver problemas en el momento. |
| Mariano | Redistribución | 2.5 h | R-Central1 propaga las estáticas al backbone y se configuran al menos dos puntos más entre OSPF, EIGRP y RIPv2. |
| Oscar | Borde de Occidente y Norte | 2 h | Protocolo acordado en R-Occidente y R-Norte y pings de extremo a extremo hacia el Data Center. |
| Luis | Etiquetas en la topología | 1 h | Marcar en Packet Tracer el área de cada protocolo de enrutamiento y verificar los hostnames. |
| Los tres | Sincronización corta, domingo en la noche | 0.5 h | Confirmar que todas las sedes convergen en el maestro y repartir lo que falte para el lunes. |

## Lunes 19, pruebas y manual

| Responsable | Tarea | Duración | Detalle |
|---|---|---|---|
| Mariano | Evidencias de ruteo | 2 h | Capturas de show ip route en los routers frontera y redacción del rol de cada protocolo y de la justificación del backbone. |
| Oscar | Falla del anillo STP | 2 h | Apagar un enlace del anillo y capturar la reconvergencia. Párrafos de justificación de Occidente y Norte y extractos de configuración. |
| Luis | Cierre del manual | 3 h | Capturas de HSRP, matriz de pings entre sedes, mapeo lógico de troncales y puertos de acceso, y revisión de redacción. |
| Los tres | Revisión final contra la rúbrica | 1.5 h | Recorrer la tabla 8.1 requisito por requisito y hacer el commit final. |

## Decisiones de la reunión de arranque

| Decisión | Propuesta inicial |
|---|---|
| Valores XX y Y | XX son los dos últimos dígitos del carné del coordinador y Y el último dígito del número de grupo. Con eso salen los IDs de VLAN (1Y, 2Y, 1Y+1, etc.) y el dominio VTP bantechXX. |
| Dominio por sede | Una opción defendible es Oriente con RIPv2 (es el banco recién comprado, la red legada), Occidente con EIGRP (expansión regional) y Norte con OSPF. Central ya tiene asignadas las estáticas. |
| Bloques IP | Reservar un bloque por sede y uno para los enlaces del backbone, para que cada quien haga su VLSM sin chocar. |
| Versión de Packet Tracer | Todos la misma versión, porque cada sede se arma en un archivo aparte y el domingo se fusionan en el .pkt maestro. |
| Convención de nombres | Hostnames como R-Occidente, SW-N-ROOT y MS1, y nombre explícito en cada VLAN. |

## Requisitos 8.1 y responsable de verificarlos

Si alguno de estos requisitos falla, la nota es cero. Por eso cada uno tiene un dueño que lo verifica el lunes.

| Requisito | Verifica |
|---|---|
| Topología conforme al enunciado | Mariano |
| RIP, OSPF, EIGRP, HSRP, EtherChannel y VTP funcionando | Mariano |
| Subnetting VLSM o FLSM correcto | Luis |
| Repositorio privado con commits propios de los tres | Mariano |
| README.md con todo lo de la sección 4.9 | Luis |
| Evidencia de pings, inter-VLAN y convergencia | Oscar |

## Forma de trabajo en el repositorio

Cada integrante sube sus propios commits desde su cuenta, porque la rúbrica pide commits propios. Lo más práctico es una carpeta por sede con su .pkt parcial y sus extractos de configuración, y el .pkt maestro en la raíz a partir del domingo.