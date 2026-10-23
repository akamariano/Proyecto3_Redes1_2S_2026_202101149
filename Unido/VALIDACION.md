# Registro de validación del maestro BanTech GT

[← Manual técnico](../README.md) · [Pendientes de entrega](#7-evidencias-por-incorporar-a-la-entrega)

| Sesión de revisión | Packet Tracer | Dispositivos | Enlaces |
|---|---|:---:|:---:|
| 23 de octubre de 2026 | 9.0.1.0858 | 112 | 120 |

Registro consolidado de respuestas reales de las herramientas de inspección y ping de Packet Tracer y salidas de consola consultadas durante la sesión. Los bloques son extractos de esas respuestas, no un volcado completo de configuración ni capturas gráficas.

> **Resultado de la sesión:** HSRP y STP respondieron en los escenarios ensayados; los canales LACP y las bases VTP se observaron operativos. El cierre conservó el uplink de MS2 apagado, por lo que sigue pendiente habilitar la segunda salida de Oriente.

## Contenido

- [Alcance](#alcance)
- [1. Conectividad](#1-conectividad)
- [2. HSRP](#2-hsrp)
- [3. Rapid PVST+](#3-rapid-pvst)
- [4. EtherChannel](#4-etherchannel)
- [5. VTP](#5-vtp)
- [6. Mapeo adicional de Oriente](#6-mapeo-adicional-de-oriente)
- [7. Evidencias por incorporar](#7-evidencias-por-incorporar-a-la-entrega)

---

## Alcance

- Archivo identificado por Packet Tracer: `Unido/BanTech_GT_v1.pkt`, ubicado fuera de esta copia del repositorio.
- Packet Tracer: 9.0.1.0858; 112 dispositivos y 120 enlaces.
- Revisión: pings representativos, HSRP ante caída del uplink de MS1, Rapid PVST+ ante caída del enlace directo de Análisis, estados LACP y coincidencia de bases VTP.
- Las capturas de `Oscar/capturas/` son evidencias de archivos parciales. Este documento registra por separado las pruebas sobre el maestro integrado.
- No se midió convergencia en milisegundos, no hubo ping continuo y no se probó pérdida completa de energía de MS1 ni caída de miembros de cada EtherChannel.

## 1. Conectividad

| Origen | IP destino | Resultado final observado | Observaciones |
|---|---|---|---|
| PC0 | 10.49.40.2, WEB1 | 4 enviados, 4 recibidos, 0 % de pérdida | La primera solicitud al canal agotó el tiempo; una ejecución posterior devolvió 4/4 |
| R-Occidente | 10.49.40.2, WEB1 | Success rate is 100 percent (5/5) | Verifica salida desde el router |
| PC-A1 | 10.49.40.2, WEB1 | 4/4 | También pasó 4/4 con el enlace directo de Análisis apagado |
| PLAT1 | 10.49.40.34, BD1 | 4/4 en la repetición | Primer ping: 2/4; también pasó 4/4 durante la prueba HSRP con respaldo disponible |
| PC0 | 10.49.10.66, PC7 | 4/4 en la repetición | Primer ping: 3/4; prueba inter-VLAN de Occidente |
| PC-A1 | 10.49.30.132, BOV1 | 4/4 en la repetición | Primer ping: 2/4 |
| BOV1 | 10.49.40.2, WEB1 | 4/4 | Durante la prueba con MS2 activo |
| PC0 | 10.49.20.2, PC-A1 | Sin resultado confirmado | La solicitud adicional agotó el tiempo del canal; no se obtuvo resumen ICMP |

Formato real devuelto en los pings completos desde hosts:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
```

La pérdida inicial seguida de una repetición completa es compatible con resolución ARP inicial, pero no se trazó cada paquete para establecer esa causa. Un timeout del canal no debe presentarse como un ping aprobado ni como pérdida ICMP confirmada. Sigue pendiente una prueba directa PLAT1 → BOV1 para documentar inter-VLAN de Oriente.

## 2. HSRP

| Etapa | Evidencia principal | Resultado |
|---|---|---|
| Estado preferido | `show standby brief` en MS1 | Activo en ambos grupos, prioridad 105 |
| Falla con respaldo disponible | Prioridad de MS1 95; MS2 activo a 100 | PLAT1 → BD1 y BOV1 → WEB1: 4/4 |
| Recuperación | MS1 vuelve a prioridad 105 y activo | `preempt` observado |
| Cierre de la sesión | Uplink de MS2 apagado | Redundancia de salida pendiente |

### Estado preferido observado

<details>
<summary>Ver consola · MS1 activo y seguimiento del uplink</summary>

```text
MS1#show standby brief
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Vl84        84   105 P Active   local           10.49.30.131    10.49.30.129
Vl94        94   105 P Active   local           10.49.30.3      10.49.30.1
```

`show standby` confirmó para el grupo 84 de MS1:

```text
Priority 105 (configured 105)
  Track interface GigabitEthernet1/0/1 state Up decrement 10
```

</details>

### Conmutación con respaldo disponible

La caída del uplink redujo a 95 la prioridad de ambos grupos de MS1. Con el uplink de MS2 habilitado temporalmente, la consola mostró:

<details>
<summary>Ver consola · Transición de MS1 y rol activo de MS2</summary>

```text
MS1#show standby brief
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Vl84        84   95  P Speak    10.49.30.131    unknown         10.49.30.129
Vl94        94   95  P Speak    10.49.30.3      unknown         10.49.30.1

MS2#show standby brief
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Vl84        84   100 P Active   local           10.49.30.131    10.49.30.129
Vl94        94   100 P Active   local           10.49.30.3      10.49.30.1
```

</details>

La salida captura un instante de transición: MS2 ya estaba activo y MS1 aún mostraba `Speak`. PLAT1 → BD1 y BOV1 → WEB1 respondieron 4/4 en esa fase de la prueba. Después de restaurar el uplink, MS1 volvió a aparecer activo en ambos grupos con prioridad 105.

### Incidencia encontrada y estado de cierre

En una primera caída de MS1 se encontró también apagado el uplink de MS2. El ping PLAT1 → BD1 recibió 1/4 respuestas. La inspección de MS2 confirmó:

```text
GigabitEthernet1/0/1 is administratively down, line protocol is down (disabled)
```

Se habilitó temporalmente ese puerto de MS2 para repetir la conmutación con una salida disponible. Al terminar se restauró el uplink de MS1 y se devolvió el de MS2 al estado apagado encontrado. El ping final PLAT1 → BD1 respondió 4/4 por la red recuperada, pero el chequeo de salud aún señaló los dos extremos del único enlace caído:

```text
CORE-3  GigabitEthernet1/0/2
MS2     GigabitEthernet1/0/1
```

> **Pendiente operativo:** la prueba validó el mecanismo HSRP condicionado a un respaldo disponible, pero el estado de cierre no ofrece dos salidas operativas.

**Acción antes de entregar:**

```cisco
! MS2: habilitar el uplink y verificar antes de guardar
interface GigabitEthernet1/0/1
 no shutdown
```

### Procedimiento para repetir

1. Comprobar ambos uplinks `up/up`, MS1 activo a 105 y MS2 en espera a 100.
2. En una copia del maestro, apagar únicamente Gi1/0/1 de MS1.
3. Consultar `show standby brief` en ambos distribuidores y repetir pings desde Plataforma y Bóveda hacia el Data Center.
4. Verificar MS2 activo en los grupos 84 y 94 y conservar las salidas de consola.
5. Restaurar Gi1/0/1 de MS1 y comprobar su recuperación como activo por `preempt`.
6. Dejar ambos uplinks habilitados y comprobar el estado final antes de guardar la copia de entrega.

## 3. Rapid PVST+

Prueba en SW-N-ANALISIS, VLAN 54. La raíz observada fue SW-N-PRINCIPAL: prioridad 24630 (24576 + 54), dirección `0002.4A17.05CB`.

| Etapa | Puerto raíz | Puerto alternativo | Costo total a la raíz |
|---|---|---|:---:|
| Inicial | Fa0/24 | Fa0/23 bloqueado | 19 |
| Fa0/24 apagado | Fa0/23 | — | 38 |
| Restaurado | Fa0/24 | Fa0/23 bloqueado | 19 |

<details>
<summary>Ver consola · Estados antes, durante y después de la falla</summary>

Extractos de `show spanning-tree vlan 54`:

```text
ANTES: costo hacia la raíz 19, puerto 24
Fa0/23           Altn BLK 19        128.23   P2p
Fa0/24           Root FWD 19        128.24   P2p

CON Fa0/24 APAGADO: costo hacia la raíz 38, puerto 23
Fa0/23           Root FWD 19        128.23   P2p

RESTAURADO: costo hacia la raíz 19, puerto 24
Fa0/23           Altn BLK 19        128.23   P2p
Fa0/24           Root FWD 19        128.24   P2p
```

</details>

El costo 19 de la fila del puerto es su costo local; 38 es el costo total del camino alternativo a la raíz. Durante la caída, PC-A1 → WEB1 recibió 4/4 respuestas. Fa0/24 quedó restaurado.

Para repetir: capturar el estado inicial, apagar Fa0/24 en SW-N-ANALISIS, capturar el nuevo puerto raíz y el ping hacia WEB1, restaurar Fa0/24 y comprobar el retorno al camino directo. Las capturas históricas de `Oscar/capturas/60_...`, `61_...` y `62_...` corresponden al parcial y pueden mostrar otro puerto alternativo.

## 4. EtherChannel

`show etherchannel summary` confirmó los siguientes extremos, todos negociados con LACP:

| Equipo | Canal | Estado | Miembros observados |
|---|---|---|---|
| CORE-1 | Po12 | RU | Gi1/1/1(P), Gi1/1/2(P) |
| CORE-1 | Po13 | RU | Gi1/1/3(P), Gi1/1/4(P) |
| CORE-2 | Po12 | RU | Gi1/1/1(P), Gi1/1/2(P) |
| CORE-2 | Po23 | RU | Gi1/1/3(P), Gi1/1/4(P) |
| CORE-3 | Po13 | RU | Gi1/1/1(P), Gi1/1/2(P) |
| CORE-3 | Po23 | RU | Gi1/1/3(P), Gi1/1/4(P) |
| MS1 | Po12 | SU | Gi1/0/2(P), Gi1/0/3(P) |
| MS2 | Po12 | SU | Gi1/0/2(P), Gi1/0/3(P) |
| MS-DC | Po1 | SU | Gi1/0/2(P), Gi1/0/3(P) |
| MS-DC | Po2 | SU | Gi1/0/4(P), Gi1/0/5(P) |
| SW-DC-ACC1 | Po1 | SU | Gi0/1(P), Gi0/2(P) |
| SW-DC-ACC2 | Po2 | SU | Gi0/1(P), Gi0/2(P) |

| Indicador | Significado |
|:---:|---|
| `R` | Canal de capa 3 |
| `S` | Canal de capa 2 |
| `U` | Canal en uso |
| `P` | Puerto agrupado en el canal |

No aparecieron miembros suspendidos o independientes en estas salidas. No se ejecutó una prueba de caída de miembro por cada canal.

## 5. VTP

Todos los equipos siguientes reportaron versión 2 y dominio `bantech49`:

| Sede | Equipo | Modo | Revisión | VLANs totales |
|---|---|---|---|---:|
| Occidente | SW-O-PRINCIPAL | Server | 9 | 9 |
| Occidente | SW-O-CAJAS | Client | 9 | 9 |
| Occidente | SW-O-ASESORES | Client | 9 | 9 |
| Occidente | SW-O-GERENCIA | Client | 9 | 9 |
| Occidente | SW-O-SEGURIDAD | Client | 9 | 9 |
| Norte | SW-N-PRINCIPAL | Server | 7 | 8 |
| Norte | SW-N-ANALISIS | Client | 7 | 8 |
| Norte | SW-N-AUDITORIA | Client | 7 | 8 |
| Norte | SW-N-LEGAL | Client | 7 | 8 |

<details>
<summary>Ver evidencia · Resúmenes MD5 de las bases VTP</summary>

Resúmenes coincidentes entre servidor y clientes dentro de cada sede:

```text
Occidente: 1B 0D 1A 48 93 34 25 B7 D9 96 5B AD C9 54 DC CB
Norte:     04 22 B8 5D 7C B1 E1 29 B9 14 17 DF FA D5 62 2C
```

</details>

Los totales incluyen las cinco VLANs de fábrica. Las sedes están separadas por capa 3 y tienen bases independientes, aunque utilicen el mismo nombre de dominio. Se verificó coincidencia de bases existentes; no se creó ni eliminó una VLAN para ensayar propagación durante esta revisión.

## 6. Mapeo adicional de Oriente

Consultas de solo lectura realizadas para consolidar el manual:

- `show vlan brief`: SW-E-ACC1 Fa0/1–7 en VLAN 94 Plataforma; SW-E-ACC2 Fa0/1–7 en VLAN 84 Boveda.
- `show interfaces trunk`: Gi0/1 y Gi0/2 en ambos switches de acceso transportan 84,94, con VLAN nativa 1.
- `show cdp neighbors` en ACC1: Gi0/1 → MS1 Gi1/0/4 y Gi0/2 → MS2 Gi1/0/4.
- `show cdp neighbors` en ACC2: Gi0/1 → MS1 Gi1/0/5 y Gi0/2 → MS2 Gi1/0/5.

## 7. Evidencias por incorporar a la entrega

- [ ] **Maestro definitivo:** `.pkt` dentro de `Unido/`, con ambos uplinks habilitados y configuración guardada.
- [ ] **Conectividad adicional:** resumen ICMP confirmado de PC0 → PC-A1 y ping directo PLAT1 → BOV1.
- [ ] **Capturas del maestro:** `show standby brief` antes/durante/después, STP antes/durante/después y pings de la matriz si la evaluación exige formato gráfico.
- [ ] **Salud final:** comprobar después de habilitar el uplink de MS2. El último chequeo registrado no encontró IPs duplicadas; sí encontró ese enlace caído.

---

[Volver al contenido](#contenido) · [Volver al manual técnico](../README.md#11-pruebas-de-conectividad-y-alta-disponibilidad)
