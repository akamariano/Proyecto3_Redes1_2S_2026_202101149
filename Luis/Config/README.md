# Configuraciones de Oriente

Responsable: Luis Pablo Carlos Tapia, carné 202203257.

Estos archivos son comandos IOS para configurar los cuatro switches de Oriente. Son plantillas de configuración del diseño documentado, no volcados completos de `running-config` ni respaldos automáticos del maestro.

| Archivo | Equipo | Función |
|---|---|---|
| [MS1.conf](MS1.conf) | MS1, 3650-24PS | HSRP preferido, raíz STP, RIPv2 y Po12 |
| [MS2.conf](MS2.conf) | MS2, 3650-24PS | Respaldo HSRP, raíz STP secundaria, RIPv2 y Po12 |
| [SW-E-ACC1.conf](SW-E-ACC1.conf) | SW-E-ACC1, 2960-24TT | Acceso de PLAT1–7, VLAN 94 |
| [SW-E-ACC2.conf](SW-E-ACC2.conf) | SW-E-ACC2, 2960-24TT | Acceso de BOV1–7, VLAN 84 |

## Aplicación

1. Usar el archivo correspondiente al hostname. En los 3650, comprobar primero la fuente de alimentación y el cableado indicado en el [README de Luis](../README.md#vlans-e-interfaces).
2. Aplicar los comandos por CLI en una copia de trabajo. Cada archivo incluye `enable`, `configure terminal`, `end` y `write memory`.
3. Verificar el estado después de converger. Las interfaces se configuran individualmente: durante la implementación se observaron problemas con `interface range` mediante `configureIosDevice`.
4. En Po12, el Port-channel y ambos miembros deben permitir `84,94`; los dos extremos usan LACP `active`.

## Parámetros esperados

| Parámetro | MS1 | MS2 |
|---|---|---|
| Gi1/0/1 hacia CORE-3 | 10.49.0.34/30 | 10.49.0.38/30 |
| VLAN 84, IP real | 10.49.30.130/26 | 10.49.30.131/26 |
| VLAN 94, IP real | 10.49.30.2/25 | 10.49.30.3/25 |
| Prioridad HSRP, grupos 84 y 94 | 105 | 100 |
| Interfaz seguida / decremento | Gi1/0/1 / 10 | Gi1/0/1 / 10 |
| Preempt | Sí | Sí |
| Prioridad STP, VLANs 84 y 94 | 24576 | 28672 |
| Po12 | Gi1/0/2 y Gi1/0/3 | Gi1/0/2 y Gi1/0/3 |

Gateways virtuales: `10.49.30.129` para Bóveda y `10.49.30.1` para Plataforma. Las SVIs son pasivas en RIPv2. Los cuatro switches usan VTP transparente y Rapid PVST+. En acceso, Gi0/1 y Gi0/2 son troncales; Fa0/1–7 tienen PortFast y BPDU Guard.

## Verificación por consola

En MS1 y MS2:

```ios
show ip interface brief
show standby brief
show etherchannel summary
show interfaces trunk
show spanning-tree vlan 84
show spanning-tree vlan 94
show ip protocols
show ip route rip
```

En SW-E-ACC1 y SW-E-ACC2:

```ios
show vlan brief
show interfaces trunk
show spanning-tree vlan 84
show spanning-tree vlan 94
```

En estado estable se espera MS1 Active y MS2 Standby en ambos grupos, Po12 `SU` con sus dos miembros `P`, y un camino STP válido desde cada switch de acceso. En CORE-3, `show ip route rip` permite comprobar las redes de Oriente.

Pruebas de host: PLAT7 a BOV7 (`10.49.30.138`), PLAT7 a BD1 (`10.49.40.34`) y retorno desde BD1 a PLAT7 (`10.49.30.10`). Los resultados y las [evidencias](../README.md#evidencias-de-redundancia) están documentados en el README.

Se configura `standby <grupo> track GigabitEthernet1/0/1` en ambos equipos. Packet Tracer 9.0.1 aplica un decremento de 10 y no acepta el decremento explícito en esa sintaxis. Por eso MS1 usa 105: al caer su uplink baja a 95 y MS2, con 100 y `preempt`, asume ambos gateways. Al restaurarlo, MS1 vuelve a 105 y recupera el rol activo automáticamente.

La corrección completa incluye el agregado de respaldo de CORE-3 descrito en [Unido/CORE.conf](../../Unido/CORE.conf). Mantiene el anuncio de retorno en OSPF cuando el simulador retira las rutas específicas redistribuidas al perder un miembro del ECMP RIP. Las rutas específicas de usuarios siguen aprendidas por RIPv2. Resultados y procedimiento reproducible: [verificación final](../../Unido/VERIFICACION_FINAL.md).
