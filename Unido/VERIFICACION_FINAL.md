# Verificación final de BanTech GT

Fecha: **23 de octubre de 2026**. Cisco Packet Tracer **9.0.1.0858**.

## Archivos de entrega

- **[BanTech_GT_v1.pkt](BanTech_GT_v1.pkt): maestro final**, con ambos uplinks restaurados.
- [Prueba_HSRP_uplink_MS1_caido.pkt](Prueba_HSRP_uplink_MS1_caido.pkt): copia de prueba, intencionalmente con Gi1/0/1 de MS1 en `shutdown`.
- [Topologia_final.png](Topologia_final.png): captura real del canvas después del ordenamiento.
- [fondo_final.svg](fondo_final.svg): fondo editable por sede y protocolo.
- [CORE.conf](CORE.conf): extractos actuales, separados por equipo, del núcleo.
- [MS1.conf](../Luis/Config/MS1.conf) y [MS2.conf](../Luis/Config/MS2.conf): comandos de Oriente actualizados.

Las copias de las carpetas personales y las capturas del 22/10 son históricas. El maestro vigente es exclusivamente el de `Unido/`.

## 1. Correcciones implementadas

### Seguimiento del uplink en HSRP

En ambos grupos de MS1 se configuró prioridad **105**, `preempt` y seguimiento de `GigabitEthernet1/0/1`. En MS2 se conserva prioridad **100**, con `preempt` y seguimiento de su propio uplink.

El IOS simulado admite `standby <grupo> track GigabitEthernet1/0/1` con decremento predeterminado **10**. La variante con decremento explícito no fue aceptada. Con prioridades 105/100, perder el uplink reduce a MS1 a **95**, por debajo de MS2; perder el uplink de MS2 reduce su prioridad a **90**.

```ios
! MS1
interface Vlan84
 standby 84 priority 105
 standby 84 preempt
 standby 84 track GigabitEthernet1/0/1
interface Vlan94
 standby 94 priority 105
 standby 94 preempt
 standby 94 track GigabitEthernet1/0/1
! MS2: prioridad 100 y el mismo seguimiento en ambos grupos
```

### Retorno OSPF durante la falla

La primera prueba con seguimiento mostró un segundo problema: CORE-3 conservaba las rutas RIP por MS2, pero CORE-1 y R-CENTRAL1 perdían las rutas específicas redistribuidas en OSPF al retirar uno de los siguientes saltos ECMP. Cambiar únicamente el estado HSRP no resolvía el retorno desde Central.

Se añadió en **CORE-3**, no en los equipos de Oriente, un agregado estático `10.49.30.0/24`, preferido por MS2 con distancia 250 y con alternativa flotante por MS1 con distancia 251. Se redistribuye en OSPF. Las rutas RIP de VLAN 94 (`/25`) y VLAN 84 (`/26`) siguen siendo más específicas y se usan normalmente. El agregado no cambia las subredes de los hosts.

```ios
ip route 10.49.30.0 255.255.255.0 10.49.0.38 250
ip route 10.49.30.0 255.255.255.0 10.49.0.34 251
router ospf 1
 redistribute rip subnets
 redistribute static subnets
```

El resumen cubre el bloque reservado de Oriente. El rango no asignado `10.49.30.192/26` permanece sin hosts. No representa una VLAN adicional.

## 2. Prueba real: uplink de MS1 caído

Se aplicó `shutdown` **solo** en Gi1/0/1 de MS1. Se esperó la conmutación y se ejecutaron pings después de converger. No se apagó el switch completo ni se reiniciaron puertos de acceso.

Extractos reales de `show standby brief`, omitiendo únicamente encabezados:

```text
MS1#show standby brief
Vl84        84   95  P Standby  10.49.30.131    local           10.49.30.129
Vl94        94   95  P Standby  10.49.30.3      local           10.49.30.1

MS2#show standby brief
Vl84        84   100 P Active   local           10.49.30.130    10.49.30.129
Vl94        94   100 P Active   local           10.49.30.2      10.49.30.1

CORE-3#show ip route static
S       10.49.30.0/24 [250/0] via 10.49.0.38
```

En CORE-1, `show ip route 10.49.30.4` resolvió la red mediante el agregado `10.49.30.0/24`, aprendido por OSPF con siguiente salto `10.49.0.6`, por Port-channel13.

| Origen | Destino | Resultado con uplink de MS1 caído |
|---|---|---|
| PLAT1, VLAN 94 | BD1, 10.49.40.34 | Enviados 4, recibidos 4, perdidos 0 |
| BOV1, VLAN 84 | BD1, 10.49.40.34 | Enviados 4, recibidos 4, perdidos 0 |
| BD1, Data Center | PLAT1, 10.49.30.4 | Enviados 4, recibidos 4, perdidos 0 |

La copia `.pkt` de falla conserva este escenario para inspeccionarlo en el simulador. Al abrirla hay que permitir que los protocolos converjan antes de repetir los pings.

## 3. Prueba real: uplink de MS2 caído

Se restauró el uplink de MS1 y se apagó solo Gi1/0/1 de MS2.

```text
MS2#show standby brief
Vl84        84   90  P Standby  10.49.30.130    local           10.49.30.129
Vl94        94   90  P Standby  10.49.30.2      local           10.49.30.1

CORE-3#show ip route static
S       10.49.30.0/24 [251/0] via 10.49.0.34

CORE-3#show ip route rip
R       10.49.30.0/25 [120/1] via 10.49.0.34, GigabitEthernet1/0/1
R       10.49.30.128/26 [120/1] via 10.49.0.34, GigabitEthernet1/0/1
```

Los extractos de rutas omiten los temporizadores variables. PLAT1 → BD1 y BOV1 → BD1 terminaron con **4/4 respuestas** en este estado.

## 4. Restauración automática

Se ejecutó `no shutdown` en los uplinks y se verificó el retorno de los roles, sin reiniciar puertos de acceso:

```text
MS1#show standby brief
Vl84        84   105 P Active   local           10.49.30.131    10.49.30.129
Vl94        94   105 P Active   local           10.49.30.3      10.49.30.1

MS2#show standby brief
Vl84        84   100 P Standby  10.49.30.130    local           10.49.30.129
Vl94        94   100 P Standby  10.49.30.2      local           10.49.30.1
```

Se guardaron las configuraciones de los equipos corregidos con `write memory`. El maestro se entrega en estado restaurado, no en el estado de falla de la copia de prueba.

También se cerró el ciclo de persistencia abriendo de nuevo el maestro guardado. Se verificó que la startup-config de MS1 conserva prioridad 105 y seguimiento en ambos grupos, y la de CORE-3 conserva las dos rutas del agregado y su redistribución. Después de converger y de los intentos iniciales con pérdidas, las confirmaciones individuales fueron:

| Origen | Destino | Resultado tras reabrir el maestro |
|---|---|---|
| PLAT1 | BD1, 10.49.40.34 | 4/4, 0% pérdida |
| BOV1 | BD1, 10.49.40.34 | 4/4, 0% pérdida |
| BD1 | PLAT1, 10.49.30.4 | 4/4, 0% pérdida |
| PC0, Occidente | BD1, 10.49.40.34 | 4/4, 0% pérdida |
| PC-A1, Norte | BD1, 10.49.40.34 | 4/4, 0% pérdida |

## 5. Alcance de los resultados

- La topología contiene **112 dispositivos y 120 enlaces**. El chequeo posterior al ordenamiento no detectó enlaces caídos ni IP duplicadas.
- Las interfaces físicas sin IP de R-Occidente y R-Norte corresponden a Router-on-a-Stick; las direcciones están en sus subinterfaces.
- Durante la auditoría previa se probaron los 84 equipos finales: 83 hacia BD1 y BD1 hacia PC0. Todos obtuvieron respuestas, pero hubo pérdidas bajo carga simultánea. Esa prueba no equivale a 84 pings individuales sin pérdida.
- Los resultados 4/4 de las tablas son pruebas reales después de converger. No se midió un tiempo exacto de recuperación ni se afirma conmutación instantánea sin pérdida de ningún paquete.
- CORE-3 continúa siendo un punto común de falla de Oriente; el diseño no promete tolerar la caída simultánea de ambos uplinks o de ese equipo.

## 6. Cómo reproducir la prueba

1. Abrir el maestro `Unido/BanTech_GT_v1.pkt` y esperar la convergencia. Para conservarlo, realizar las pruebas sobre una copia.
2. En MS1 y MS2: `show standby brief` y `show standby`. Comprobar prioridades 105/100 y seguimiento del uplink con decremento 10.
3. En MS1:

   ```ios
   configure terminal
   interface GigabitEthernet1/0/1
   shutdown
   end
   ```

4. Esperar la conmutación; comprobar MS1 Standby/95 y MS2 Active/100.
5. Desde PLAT1 y BOV1: `ping 10.49.40.34`. Desde BD1: `ping 10.49.30.4`.
6. En CORE-3: `show ip route static` y `show ip route rip`. En CORE-1: `show ip route 10.49.30.4`.
7. Restaurar Gi1/0/1 de MS1 con `no shutdown`; comprobar el retorno a Active/105 y repetir los pings.
8. Para la prueba simétrica, mantener MS1 operativo y repetir el procedimiento apagando únicamente el uplink de MS2. Comprobar el agregado con distancia 251 y restaurar al finalizar.

No guardar el maestro con interfaces apagadas. Las capturas antiguas con pings fallidos permanecen como historial y no son la evidencia de esta corrección.
