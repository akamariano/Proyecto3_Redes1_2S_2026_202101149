# Sede Oriente — Centro Financiero Legado

Responsable: Luis. Implementación y verificación: 20 de octubre de 2026.

La simulación integrada en progreso está en [Unido/BanTech_GT_v1.pkt](../Unido/BanTech_GT_v1.pkt). Incluye el backbone, Central y Data Center de Mariano y la Sede Oriente. La integración de Occidente y Norte sigue pendiente.

## Arquitectura

![Backbone, Data Center y Oriente](../Images/oriente-backbone-integrado.png)

MS1 y MS2 son switches multicapa 3650-24PS, cada uno con una fuente `AC-POWER-SUPPLY` en la ranura interna `2/4`. Ambos realizan enrutamiento inter-VLAN y se unen mediante un EtherChannel troncal LACP de dos cables (Po12). SW-E-ACC1 y SW-E-ACC2 son 2960-24TT y tienen un enlace troncal hacia cada equipo de distribución. Cada switch de acceso conecta siete PCs.

HSRP mantiene gateways virtuales estables para los hosts. MS1 tiene prioridad 110 y MS2 prioridad 100; ambos usan `preempt`. Rapid PVST+ evita los bucles de capa 2 de los enlaces redundantes. MS1 tiene prioridad STP 24576 para las dos VLANs y MS2 28672, alineando el gateway activo y la raíz preferida. Los puertos hacia PCs usan PortFast y BPDU Guard.

Oriente se integra con CORE-3 mediante RIPv2 y dos enlaces ruteados independientes. CORE-3 realiza la redistribución hacia OSPF. No se añadieron rutas estáticas en Oriente. Las SVIs son pasivas en RIP: sus redes se anuncian por los enlaces ruteados sin enviar actualizaciones a los hosts. Los cuatro switches usan VTP transparente y definen las VLANs localmente.

La redundancia cubre un equipo de distribución o un miembro de Po12. CORE-3 sigue siendo el punto común de salida de Oriente; cada PC tiene una única conexión a su switch de acceso.

## VLANs e interfaces

| VLAN | Nombre | Red | Gateway virtual | IP MS1 | IP MS2 |
|---|---|---|---|---|---|
| 84 | Boveda | 10.49.30.128/26 | 10.49.30.129 | 10.49.30.130 | 10.49.30.131 |
| 94 | Plataforma | 10.49.30.0/25 | 10.49.30.1 | 10.49.30.2 | 10.49.30.3 |

| Enlace | Extremo A | Extremo B | Red / VLANs |
|---|---|---|---|
| CORE-3 a MS1 | CORE-3 Gi1/0/1, 10.49.0.33 | MS1 Gi1/0/1, 10.49.0.34 | 10.49.0.32/30 |
| CORE-3 a MS2 | CORE-3 Gi1/0/2, 10.49.0.37 | MS2 Gi1/0/1, 10.49.0.38 | 10.49.0.36/30 |
| Po12, LACP | MS1 Gi1/0/2-3 | MS2 Gi1/0/2-3 | Troncal 84,94 |
| Acceso Plataforma | MS1 Gi1/0/4 | SW-E-ACC1 Gi0/1 | Troncal 84,94 |
| Acceso Plataforma, respaldo | MS2 Gi1/0/4 | SW-E-ACC1 Gi0/2 | Troncal 84,94 |
| Acceso Bóveda | MS1 Gi1/0/5 | SW-E-ACC2 Gi0/1 | Troncal 84,94 |
| Acceso Bóveda, respaldo | MS2 Gi1/0/5 | SW-E-ACC2 Gi0/2 | Troncal 84,94 |

Todos los enlaces nuevos usan cobre directo; los equipos negociaron los enlaces correctamente en Packet Tracer. Cada PC usa el puerto Fa0/1 a Fa0/7 de su switch según su número.

## Equipos finales

| Equipo | Switch / puerto | VLAN | IP | Máscara | Gateway |
|---|---|---|---|---|---|
| PLAT1 | SW-E-ACC1 Fa0/1 | 94 | 10.49.30.4 | 255.255.255.128 | 10.49.30.1 |
| PLAT2 | SW-E-ACC1 Fa0/2 | 94 | 10.49.30.5 | 255.255.255.128 | 10.49.30.1 |
| PLAT3 | SW-E-ACC1 Fa0/3 | 94 | 10.49.30.6 | 255.255.255.128 | 10.49.30.1 |
| PLAT4 | SW-E-ACC1 Fa0/4 | 94 | 10.49.30.7 | 255.255.255.128 | 10.49.30.1 |
| PLAT5 | SW-E-ACC1 Fa0/5 | 94 | 10.49.30.8 | 255.255.255.128 | 10.49.30.1 |
| PLAT6 | SW-E-ACC1 Fa0/6 | 94 | 10.49.30.9 | 255.255.255.128 | 10.49.30.1 |
| PLAT7 | SW-E-ACC1 Fa0/7 | 94 | 10.49.30.10 | 255.255.255.128 | 10.49.30.1 |
| BOV1 | SW-E-ACC2 Fa0/1 | 84 | 10.49.30.132 | 255.255.255.192 | 10.49.30.129 |
| BOV2 | SW-E-ACC2 Fa0/2 | 84 | 10.49.30.133 | 255.255.255.192 | 10.49.30.129 |
| BOV3 | SW-E-ACC2 Fa0/3 | 84 | 10.49.30.134 | 255.255.255.192 | 10.49.30.129 |
| BOV4 | SW-E-ACC2 Fa0/4 | 84 | 10.49.30.135 | 255.255.255.192 | 10.49.30.129 |
| BOV5 | SW-E-ACC2 Fa0/5 | 84 | 10.49.30.136 | 255.255.255.192 | 10.49.30.129 |
| BOV6 | SW-E-ACC2 Fa0/6 | 84 | 10.49.30.137 | 255.255.255.192 | 10.49.30.129 |
| BOV7 | SW-E-ACC2 Fa0/7 | 84 | 10.49.30.138 | 255.255.255.192 | 10.49.30.129 |

Las subredes conservan capacidad para los 60 hosts de Plataforma y 40 de Bóveda del plan; se simulan siete PCs por VLAN.

## Configuraciones

- [MS1.conf](MS1.conf)
- [MS2.conf](MS2.conf)
- [SW-E-ACC1.conf](SW-E-ACC1.conf)
- [SW-E-ACC2.conf](SW-E-ACC2.conf)

Son comandos de configuración para pegar en la CLI IOS. Durante la automatización, `configureIosDevice` no aplicó correctamente los rangos de interfaces; se resolvió configurando cada interfaz individualmente y verificando por consola. En Po12 deben coincidir las VLANs permitidas del Port-channel y sus miembros antes de agruparlos.

## Verificaciones reales

### Estado estable

`show standby brief` mostró MS1 Active y MS2 Standby en los grupos 84 y 94, con las IPs virtuales 10.49.30.129 y 10.49.30.1. MS1 reconoció como standby 10.49.30.131 y 10.49.30.3 respectivamente.

`show etherchannel summary` mostró en ambos equipos:

```text
12     Po12(SU)           LACP   Gig1/0/2(P) Gig1/0/3(P)
```

`show ip route rip` en CORE-3 mostró ambas redes por los dos equipos:

```text
R 10.49.30.0/25 [120/1] via 10.49.0.34, GigabitEthernet1/0/1
               [120/1] via 10.49.0.38, GigabitEthernet1/0/2
R 10.49.30.128/26 [120/1] via 10.49.0.34, GigabitEthernet1/0/1
                 [120/1] via 10.49.0.38, GigabitEthernet1/0/2
```

| Origen | Destino | Resultado final |
|---|---|---|
| PLAT7 | BOV7, 10.49.30.138 | 4 enviados, 4 recibidos |
| PLAT1 | BD1, 10.49.40.34 | 4 enviados, 4 recibidos |
| BOV1 | WEB1, 10.49.40.2 | 4 enviados, 4 recibidos |
| BD1 | PLAT1, 10.49.30.4 | 4 enviados, 4 recibidos tras recuperar MS1 |

Algunos primeros pings tuvieron pérdidas durante el establecimiento inicial; las repeticiones indicadas terminaron con 0% de pérdida.

### Falla completa de MS1

Se apagó MS1 y se consultó `show standby brief` en MS2. Ambos grupos pasaron a Active local. Durante la reconvergencia se observaron pings con 100% y luego 25% de pérdida. Después de converger:

- BOV7 → BD1: 4/4 respuestas con MS1 apagado.
- PLAT1 → BOV1: 4/4 respuestas con MS1 apagado.

Se encendió MS1, se verificó su retorno a Active mediante `preempt` y el retorno de MS2 a Standby. La recuperación también tuvo pérdidas transitorias; no se afirma conmutación sin pérdida ni se midió un tiempo exacto de recuperación.

### Falla de un miembro de Po12

Se aplicó `shutdown` a Gi1/0/2 de MS1. Po12 permaneció `SU`, con Gi1/0/2(D) y Gi1/0/3(P). PLAT7 → BOV7 tuvo inicialmente 2/4 respuestas y después 4/4. Se restauró la interfaz con `no shutdown`.

Estas pruebas validan la falla completa del switch y la de un miembro del canal. No se configuró seguimiento del uplink para HSRP: la caída aislada de Gi1/0/1 no reduce por sí sola la prioridad del grupo.

## Pendientes de integración

- Incorporar las sedes Occidente y Norte de Oscar.
- Completar la matriz de pings entre las cuatro sedes.
- Agregar capturas de consola y topología al manual consolidado.
