# Backbone Core, Data Center y Sede Central

Redes de Computadoras 1, Proyecto 2, BanTech GT. Grupo 4.

Responsable: Mariano Roberto Rac Noguera, carné 202101149, coordinador del grupo. Este documento detalla la parte del proyecto que diseñé, configuré y probé. El manual consolidado está en el [README de la raíz](../README.md) y el maestro final en [Unido/BanTech_GT_v1.pkt](../Unido/BanTech_GT_v1.pkt). Los `.conf` y el `.pkt` de esta carpeta conservan la entrega parcial histórica; los extractos actuales del núcleo y las correcciones del 23/10 están en [Unido/CORE.conf](../Unido/CORE.conf) y en la [verificación final](../Unido/VERIFICACION_FINAL.md).

## 1. Resumen de la responsabilidad

Mi parte del proyecto es el núcleo de la red y el segmento de alta seguridad del banco. Eso incluye los tres switches multicapa del backbone (CORE-1, CORE-2 y CORE-3), el router de borde WAN (R-WAN), los dos routers de la Sede Central (R-CENTRAL1 y R-CENTRAL2) y todo el Data Center (MS-DC, SW-DC-ACC1, SW-DC-ACC2, catorce servidores y siete estaciones del NOC). También dejé configurados los tres puntos de redistribución del proyecto (R-WAN, CORE-3 y R-CENTRAL1) y mi extremo de los cuatro enlaces que conectan el backbone con las sedes de Oscar y Luis, de modo que al fusionar los archivos en el .pkt maestro la red converja sin tocar el núcleo.

Los parámetros de identificación del grupo son XX = 49 (los dos últimos dígitos del carné del coordinador) e Y = 4 (grupo 4). Con ellos el dominio VTP del proyecto es `bantech49` y todo el direccionamiento usa el bloque `10.49.0.0/16`, con un /24 por sede: 10.49.0.0/24 para el backbone, 10.49.10.0/24 para Occidente, 10.49.20.0/24 para Norte, 10.49.30.0/24 para Oriente y 10.49.40.0/24 para la Sede Central y el Data Center.

## 2. Diseño del backbone

![Boceto del backbone](../Images/boceto_backbone.png)

El boceto muestra el núcleo en triángulo dentro del dominio OSPF área 0, el dominio EIGRP 10 de Occidente a la izquierda, el dominio RIPv2 de Oriente a la derecha y el segmento de rutas estáticas de la Sede Central abajo, con la red /30 de cada enlace.

### 2.1 Núcleo en triángulo

El núcleo está formado por tres switches 3650-24PS conectados entre sí en triángulo, de modo que cada switch tiene un enlace directo hacia los otros dos. La alternativa más barata sería una línea o una estrella con un switch central, pero en ese caso la caída del switch central o de un solo enlace parte la red en dos. En el triángulo, si cae cualquier enlace o cualquier switch, entre los dos equipos restantes siempre queda un camino, ya sea directo o a través del tercero. Se eligió porque es la forma más pequeña que cumple el requisito del enunciado de tener al menos tres equipos de capa 3 en el núcleo sin dejar un punto único de falla.

### 2.2 EtherChannel de capa 3 con LACP

Cada lado del triángulo es un EtherChannel de dos enlaces de fibra (módulos GLC-LH-SMD en los puertos Gi1/1/1 a Gi1/1/4). Los dos enlaces físicos se comportan como una sola interfaz lógica, el Port-channel, que suma el ancho de banda de ambos y sigue funcionando si uno de ellos cae, sin que OSPF tenga que recalcular rutas. Cada Port-channel se nombró por los dos switches que une (Po12 entre CORE-1 y CORE-2, Po13 entre CORE-1 y CORE-3, Po23 entre CORE-2 y CORE-3), lo que permite saber qué enlace es con solo leer el nombre de la interfaz.

Los canales son de capa 3, es decir, los puertos físicos y el Port-channel llevan `no switchport` y la dirección IP va directamente sobre el Port-channel. La alternativa habitual es un EtherChannel troncal de capa 2 con SVIs en cada switch. Con capa 2, los tres switches forman un bucle que Spanning Tree tiene que cortar bloqueando uno de los lados, con lo cual un tercio del núcleo queda sin uso y la reconvergencia depende de STP. Con capa 3 cada lado es una red /30 independiente, no hay Spanning Tree en el núcleo, los tres lados se usan al mismo tiempo y la redundancia la resuelve OSPF por costo, incluso con balanceo de carga entre caminos de igual costo.

La negociación se hace con LACP en modo `active` en ambos extremos. LACP es el estándar IEEE 802.3ad, a diferencia de PAgP, que es propietario de Cisco, y a diferencia del modo `on`, que forma el canal sin negociar y no detecta si el otro extremo está mal configurado. Con ambos extremos en `active` el canal se forma sin depender de cuál de los dos inicia la negociación.

### 2.3 OSPF en el núcleo

El núcleo corre OSPF proceso 1 en el área 0. OSPF es un estándar abierto, converge rápido porque cada router conoce la topología completa del área y puede crecer por áreas si el banco agrega más sedes. EIGRP habría funcionado igual de bien en un laboratorio con solo equipo Cisco, pero el enunciado reserva EIGRP para la expansión regional, y RIPv2 no sirve como núcleo porque su métrica de saltos no distingue un enlace de fibra agregado de uno de cobre y converge más lento. A cada equipo OSPF se le fijó un router ID fácil de reconocer en las tablas de vecinos.

| Equipo | Router ID |
|---|---|
| CORE-1 | 1.1.1.1 |
| CORE-2 | 2.2.2.2 |
| CORE-3 | 3.3.3.3 |
| R-WAN | 4.4.4.4 |
| R-CENTRAL1 | 5.5.5.5 |

### 2.4 Ubicación de cada dominio de enrutamiento

Cada dominio se colgó de un switch distinto del núcleo, de forma que la carga de redistribución queda repartida y la falla de un switch del núcleo solo afecta a la sede que depende de él. Occidente, que representa la expansión regional, usa EIGRP 10 y llega por un enlace serial WAN a R-WAN, que a su vez se conecta a CORE-2. Se usa un router 2911 con HWIC-2T en lugar de llevar el serial directo al núcleo porque los 3650 no tienen interfaces seriales, y además así el borde WAN queda en un equipo dedicado. Norte, el centro de autorización de créditos, habla OSPF área 0 directamente con CORE-2, porque es una sede crítica y conviene que no dependa de ningún punto de redistribución. Oriente, el banco recién adquirido, usa RIPv2 como red legada y sube con MS1 y MS2 a CORE-3. La Sede Central y el Data Center quedan detrás de R-CENTRAL1, conectado a CORE-1, en un segmento de rutas estáticas por la política de aislamiento del enunciado.

### 2.5 Un solo punto de redistribución por cada par de dominios

Entre cada par de dominios hay exactamente un equipo que redistribuye: R-WAN entre OSPF y EIGRP 10, CORE-3 entre OSPF y RIPv2, y R-CENTRAL1 de estáticas hacia OSPF. Cuando dos equipos redistribuyen entre los mismos dos protocolos, una ruta puede salir de un dominio por un equipo y regresar por el otro como si fuera externa, lo que produce bucles de enrutamiento o caminos subóptimos que solo se evitan con filtros y etiquetas de ruta. Con un único punto por par no hace falta ningún filtro y el camino de cada ruta es predecible. Por esa razón MS1 y MS2 suben ambos a CORE-3 en lugar de repartirse entre dos switches del núcleo.

R-CENTRAL1 además tiene `passive-interface` en la interfaz hacia R-CENTRAL2. El enlace 10.49.0.16/30 se anuncia en OSPF, pero R-CENTRAL1 no envía hellos por esa interfaz, así que ningún protocolo dinámico entra al segmento de alta seguridad. Hacia el núcleo, el Data Center se anuncia con una sola ruta estática resumida, 10.49.40.0/24, en lugar de las tres subredes de servidores, lo que oculta la estructura interna del Data Center y deja una sola ruta en la tabla de cada equipo del núcleo.

## 3. Equipos, módulos y cableado

### 3.1 Equipos y módulos

| Equipo | Modelo | Módulos | Función |
|---|---|---|---|
| CORE-1 | 3650-24PS | AC-POWER-SUPPLY, 4 GLC-LH-SMD | Núcleo, enlace hacia la Sede Central |
| CORE-2 | 3650-24PS | AC-POWER-SUPPLY, 4 GLC-LH-SMD | Núcleo, agrega la WAN y Norte |
| CORE-3 | 3650-24PS | AC-POWER-SUPPLY, 4 GLC-LH-SMD | Núcleo y redistribución hacia RIPv2 |
| R-WAN | 2911 | HWIC-2T | Borde WAN y redistribución hacia EIGRP 10 |
| R-CENTRAL1 | 2911 | | Redistribución del segmento estático |
| R-CENTRAL2 | 2911 | | Router de acceso del Data Center |
| MS-DC | 3650-24PS | AC-POWER-SUPPLY | Distribución e inter-VLAN del Data Center |
| SW-DC-ACC1 | 2960-24TT | | Acceso de servidores BD y WEB |
| SW-DC-ACC2 | 2960-24TT | | Acceso del NOC |
| BD1 a BD7 | Server-PT | | Servidores de base de datos |
| WEB1 a WEB7 | Server-PT | | Servidores de banca virtual |
| NOC1 a NOC7 | PC-PT | | Estaciones de monitoreo |

Los 3650 no encienden sin el módulo AC-POWER-SUPPLY. El HWIC-2T de R-WAN quedó en la ranura 3, por eso su puerto serial es Serial0/3/0.

![Equipos colocados con sus nombres por defecto](../Images/Screenshot%202026-10-18%20at%208.33.07%E2%80%AFPM.png)

La primera captura muestra los equipos recién colocados en mi archivo parcial, sobre el área azul del backbone (OSPF área 0) y el área morada del Data Center y la Sede Central (rutas estáticas). Los switches y routers ya tienen su nombre, pero los servidores y las estaciones todavía conservan los nombres por defecto de Packet Tracer (Server0 a Server13 y PC0 a PC6).

![Distribución de equipos en el archivo parcial](../Images/Screenshot%202026-10-18%20at%208.36.44%E2%80%AFPM.png)

La segunda captura muestra la misma distribución antes del cableado, ya con cada equipo final renombrado según su función (BD1 a BD7, WEB1 a WEB7 y NOC1 a NOC7), que son los nombres que se usan en las tablas de direccionamiento y en las pruebas.

### 3.2 Cableado puerto por puerto

| Equipo origen | Puerto origen | Equipo destino | Puerto destino | Cable | Agrupación |
|---|---|---|---|---|---|
| CORE-1 | Gi1/1/1 | CORE-2 | Gi1/1/1 | Fibra | Po12 |
| CORE-1 | Gi1/1/2 | CORE-2 | Gi1/1/2 | Fibra | Po12 |
| CORE-1 | Gi1/1/3 | CORE-3 | Gi1/1/1 | Fibra | Po13 |
| CORE-1 | Gi1/1/4 | CORE-3 | Gi1/1/2 | Fibra | Po13 |
| CORE-2 | Gi1/1/3 | CORE-3 | Gi1/1/3 | Fibra | Po23 |
| CORE-2 | Gi1/1/4 | CORE-3 | Gi1/1/4 | Fibra | Po23 |
| CORE-1 | Gi1/0/1 | R-CENTRAL1 | Gi0/0 | Cobre directo | |
| CORE-2 | Gi1/0/1 | R-WAN | Gi0/0 | Cobre directo | |
| R-CENTRAL1 | Gi0/1 | R-CENTRAL2 | Gi0/0 | Cobre directo | |
| R-CENTRAL2 | Gi0/1 | MS-DC | Gi1/0/1 | Cobre directo | |
| MS-DC | Gi1/0/2 | SW-DC-ACC1 | Gi0/1 | Cobre directo | Po1 |
| MS-DC | Gi1/0/3 | SW-DC-ACC1 | Gi0/2 | Cobre directo | Po1 |
| MS-DC | Gi1/0/4 | SW-DC-ACC2 | Gi0/1 | Cobre directo | Po2 |
| MS-DC | Gi1/0/5 | SW-DC-ACC2 | Gi0/2 | Cobre directo | Po2 |
| CORE-2 | Gi1/0/2 | R-Norte | Gi0/0 | Cobre directo | Se cablea en la fusión |
| CORE-3 | Gi1/0/1 | MS1 | Gi1/0/1 | Cobre directo | Se cablea en la fusión |
| CORE-3 | Gi1/0/2 | MS2 | Gi1/0/1 | Cobre directo | Se cablea en la fusión |
| R-WAN | Se0/3/0 (DCE) | R-Occidente | Serial (según su módulo) | Serial DCE | Se cablea en la fusión |

Los equipos finales van en el puerto del switch de acceso con su mismo número, como se detalla en la tabla de la sección 4.3.

![Data Center cableado](../Images/Screenshot%202026-10-20%20at%2012.41.08%E2%80%AFPM.png)

La captura muestra el Data Center ya cableado: R-CENTRAL1 sube por Gi0/0 hacia CORE-1, R-CENTRAL2 baja por Gi0/1 hacia MS-DC y MS-DC llega a cada switch de acceso con dos cables (Gi1/0/2 y Gi1/0/3 hacia SW-DC-ACC1, Gi1/0/4 y Gi1/0/5 hacia SW-DC-ACC2). El panel de PDUs que aparece abajo corresponde a una primera ejecución de las pruebas y se explica en la sección 8.4.

## 4. Direccionamiento

### 4.1 Tabla de direcciones del backbone

El bloque 10.49.0.0/24 se divide en subredes /30, una por enlace punto a punto. Se ocupan las diez primeras y el resto queda libre para crecimiento.

| Enlace | Red | Extremo A | Extremo B |
|---|---|---|---|
| Po12 (CORE-1 Gi1/1/1-2, CORE-2 Gi1/1/1-2) | 10.49.0.0/30 | CORE-1 .1 | CORE-2 .2 |
| Po13 (CORE-1 Gi1/1/3-4, CORE-3 Gi1/1/1-2) | 10.49.0.4/30 | CORE-1 .5 | CORE-3 .6 |
| Po23 (CORE-2 Gi1/1/3-4, CORE-3 Gi1/1/3-4) | 10.49.0.8/30 | CORE-2 .9 | CORE-3 .10 |
| CORE-1 Gi1/0/1 a R-CENTRAL1 Gi0/0 | 10.49.0.12/30 | CORE-1 .13 | R-CENTRAL1 .14 |
| R-CENTRAL1 Gi0/1 a R-CENTRAL2 Gi0/0 | 10.49.0.16/30 | R-CENTRAL1 .17 | R-CENTRAL2 .18 |
| CORE-2 Gi1/0/1 a R-WAN Gi0/0 | 10.49.0.20/30 | CORE-2 .21 | R-WAN .22 |
| R-WAN Se0/3/0 a R-Occidente (serial) | 10.49.0.24/30 | R-WAN .25 | R-Occidente .26 |
| CORE-2 Gi1/0/2 a R-Norte Gi0/0 | 10.49.0.28/30 | CORE-2 .29 | R-Norte .30 |
| CORE-3 Gi1/0/1 a MS1 Gi1/0/1 | 10.49.0.32/30 | CORE-3 .33 | MS1 .34 |
| CORE-3 Gi1/0/2 a MS2 Gi1/0/1 | 10.49.0.36/30 | CORE-3 .37 | MS2 .38 |

Los enlaces hacia R-Occidente, R-Norte, MS1 y MS2 se cablean en la fusión del .pkt maestro, que hace Luis. Mi extremo de cada uno ya está configurado.

### 4.2 IPs de interfaces del Data Center y la Sede Central

| Equipo | Interfaz | IP | Máscara | Descripción |
|---|---|---|---|---|
| R-CENTRAL1 | Gi0/0 | 10.49.0.14 | 255.255.255.252 | Hacia CORE-1 |
| R-CENTRAL1 | Gi0/1 | 10.49.0.17 | 255.255.255.252 | Hacia R-CENTRAL2 |
| R-CENTRAL2 | Gi0/0 | 10.49.0.18 | 255.255.255.252 | Hacia R-CENTRAL1 |
| R-CENTRAL2 | Gi0/1 | 10.49.40.81 | 255.255.255.252 | Hacia MS-DC |
| MS-DC | Gi1/0/1 | 10.49.40.82 | 255.255.255.252 | Puerto ruteado hacia R-CENTRAL2 |
| MS-DC | Vlan15 | 10.49.40.33 | 255.255.255.224 | Gateway Core_BD |
| MS-DC | Vlan25 | 10.49.40.1 | 255.255.255.224 | Gateway Web_Apps |
| MS-DC | Vlan35 | 10.49.40.65 | 255.255.255.240 | Gateway NOC |

### 4.3 IPs de los equipos finales

| Equipo | Tipo | Switch y puerto | VLAN | IP | Máscara | Gateway |
|---|---|---|---|---|---|---|
| BD1 | Server-PT | SW-DC-ACC1 Fa0/1 | 15 | 10.49.40.34 | 255.255.255.224 | 10.49.40.33 |
| BD2 | Server-PT | SW-DC-ACC1 Fa0/2 | 15 | 10.49.40.35 | 255.255.255.224 | 10.49.40.33 |
| BD3 | Server-PT | SW-DC-ACC1 Fa0/3 | 15 | 10.49.40.36 | 255.255.255.224 | 10.49.40.33 |
| BD4 | Server-PT | SW-DC-ACC1 Fa0/4 | 15 | 10.49.40.37 | 255.255.255.224 | 10.49.40.33 |
| BD5 | Server-PT | SW-DC-ACC1 Fa0/5 | 15 | 10.49.40.38 | 255.255.255.224 | 10.49.40.33 |
| BD6 | Server-PT | SW-DC-ACC1 Fa0/6 | 15 | 10.49.40.39 | 255.255.255.224 | 10.49.40.33 |
| BD7 | Server-PT | SW-DC-ACC1 Fa0/7 | 15 | 10.49.40.40 | 255.255.255.224 | 10.49.40.33 |
| WEB1 | Server-PT | SW-DC-ACC1 Fa0/8 | 25 | 10.49.40.2 | 255.255.255.224 | 10.49.40.1 |
| WEB2 | Server-PT | SW-DC-ACC1 Fa0/9 | 25 | 10.49.40.3 | 255.255.255.224 | 10.49.40.1 |
| WEB3 | Server-PT | SW-DC-ACC1 Fa0/10 | 25 | 10.49.40.4 | 255.255.255.224 | 10.49.40.1 |
| WEB4 | Server-PT | SW-DC-ACC1 Fa0/11 | 25 | 10.49.40.5 | 255.255.255.224 | 10.49.40.1 |
| WEB5 | Server-PT | SW-DC-ACC1 Fa0/12 | 25 | 10.49.40.6 | 255.255.255.224 | 10.49.40.1 |
| WEB6 | Server-PT | SW-DC-ACC1 Fa0/13 | 25 | 10.49.40.7 | 255.255.255.224 | 10.49.40.1 |
| WEB7 | Server-PT | SW-DC-ACC1 Fa0/14 | 25 | 10.49.40.8 | 255.255.255.224 | 10.49.40.1 |
| NOC1 | PC-PT | SW-DC-ACC2 Fa0/1 | 35 | 10.49.40.66 | 255.255.255.240 | 10.49.40.65 |
| NOC2 | PC-PT | SW-DC-ACC2 Fa0/2 | 35 | 10.49.40.67 | 255.255.255.240 | 10.49.40.65 |
| NOC3 | PC-PT | SW-DC-ACC2 Fa0/3 | 35 | 10.49.40.68 | 255.255.255.240 | 10.49.40.65 |
| NOC4 | PC-PT | SW-DC-ACC2 Fa0/4 | 35 | 10.49.40.69 | 255.255.255.240 | 10.49.40.65 |
| NOC5 | PC-PT | SW-DC-ACC2 Fa0/5 | 35 | 10.49.40.70 | 255.255.255.240 | 10.49.40.65 |
| NOC6 | PC-PT | SW-DC-ACC2 Fa0/6 | 35 | 10.49.40.71 | 255.255.255.240 | 10.49.40.65 |
| NOC7 | PC-PT | SW-DC-ACC2 Fa0/7 | 35 | 10.49.40.72 | 255.255.255.240 | 10.49.40.65 |

## 5. VLANs y mapeo lógico del Data Center

### 5.1 VLANs del Data Center

Los IDs salen de las fórmulas del enunciado con Y = 4. Los nombres van sin tilde para evitar problemas en IOS.

| ID | Fórmula | Nombre | Red | Gateway |
|---|---|---|---|---|
| 15 | 1Y+1 | Core_BD | 10.49.40.32/27 | 10.49.40.33 |
| 25 | 2Y+1 | Web_Apps | 10.49.40.0/27 | 10.49.40.1 |
| 35 | 3Y+1 | NOC | 10.49.40.64/28 | 10.49.40.65 |

El Data Center no usa VTP. Cada switch define localmente las VLANs que necesita, con lo que una configuración errónea en un switch de otra sede no puede borrar ni modificar las VLANs del segmento de alta seguridad.

### 5.2 Mapeo lógico

| Equipo | Función | Troncales 802.1Q | Puertos de acceso |
|---|---|---|---|
| CORE-1, CORE-2, CORE-3 | Núcleo enrutado | Ninguno (Port-channel de capa 3 y puertos ruteados) | Ninguno |
| R-CENTRAL1 | Redistribución del segmento estático | Ninguno | Ninguno |
| R-CENTRAL2 | Router de acceso del Data Center | Ninguno | Ninguno |
| MS-DC | Switch principal de distribución e inter-VLAN | Po1 (Gi1/0/2-3) y Po2 (Gi1/0/4-5) | Ninguno |
| SW-DC-ACC1 | Acceso de servidores | Po1 (Gi0/1-2) | Fa0/1-7 VLAN 15, Fa0/8-14 VLAN 25 |
| SW-DC-ACC2 | Acceso del NOC | Po2 (Gi0/1-2) | Fa0/1-7 VLAN 35 |

MS-DC hace el enrutamiento inter-VLAN con SVIs y sube a cada switch de acceso por un EtherChannel troncal de capa 2 negociado con LACP. A diferencia del núcleo, aquí el canal es de capa 2 porque tiene que transportar varias VLANs hacia los servidores. Los dos cables suman capacidad para las ráfagas de tráfico transaccional y toleran la caída de uno sin cortar el servicio. Los servidores de base de datos y de banca virtual quedan en SW-DC-ACC1 y el NOC en SW-DC-ACC2, de modo que el tráfico de monitoreo no compite con el de los servidores en el mismo canal.

## 6. Extractos de configuración

Los extractos se tomaron de los archivos `.conf` de esta carpeta y muestran solo las partes importantes de cada equipo.

### 6.1 CORE-1

Fuente: [CORE-1.conf](CORE-1.conf).

```
hostname CORE-1
ip routing
!
! Po12 toward CORE-2
interface range GigabitEthernet1/1/1 - 2
 no switchport
 channel-group 12 mode active
interface Port-channel12
 no switchport
 ip address 10.49.0.1 255.255.255.252
!
! Po13 toward CORE-3
interface range GigabitEthernet1/1/3 - 4
 no switchport
 channel-group 13 mode active
interface Port-channel13
 no switchport
 ip address 10.49.0.5 255.255.255.252
!
! Routed link toward R-CENTRAL1
interface GigabitEthernet1/0/1
 no switchport
 ip address 10.49.0.13 255.255.255.252
!
router ospf 1
 router-id 1.1.1.1
 network 10.49.0.0 0.0.0.3 area 0
 network 10.49.0.4 0.0.0.3 area 0
 network 10.49.0.12 0.0.0.3 area 0
```

### 6.2 CORE-2

Fuente: [CORE-2.conf](CORE-2.conf).

```
hostname CORE-2
ip routing
!
! Po12 toward CORE-1
interface range GigabitEthernet1/1/1 - 2
 no switchport
 channel-group 12 mode active
interface Port-channel12
 no switchport
 ip address 10.49.0.2 255.255.255.252
!
! Po23 toward CORE-3
interface range GigabitEthernet1/1/3 - 4
 no switchport
 channel-group 23 mode active
interface Port-channel23
 no switchport
 ip address 10.49.0.9 255.255.255.252
!
! Routed link toward R-WAN
interface GigabitEthernet1/0/1
 no switchport
 ip address 10.49.0.21 255.255.255.252
!
! Routed link toward R-Norte (cabled at merge time)
interface GigabitEthernet1/0/2
 no switchport
 ip address 10.49.0.29 255.255.255.252
!
router ospf 1
 router-id 2.2.2.2
 network 10.49.0.0 0.0.0.3 area 0
 network 10.49.0.8 0.0.0.3 area 0
 network 10.49.0.20 0.0.0.3 area 0
 network 10.49.0.28 0.0.0.3 area 0
```

### 6.3 CORE-3

Fuentes: [CORE-3.conf](CORE-3.conf) para las interfaces y OSPF, y [CORE-3-v2.conf](CORE-3-v2.conf) para RIPv2 y la redistribución.

```
hostname CORE-3
ip routing
!
! Po13 toward CORE-1
interface range GigabitEthernet1/1/1 - 2
 no switchport
 channel-group 13 mode active
interface Port-channel13
 no switchport
 ip address 10.49.0.6 255.255.255.252
!
! Po23 toward CORE-2
interface range GigabitEthernet1/1/3 - 4
 no switchport
 channel-group 23 mode active
interface Port-channel23
 no switchport
 ip address 10.49.0.10 255.255.255.252
!
! Routed links toward MS1 and MS2 (cabled at merge time, RIPv2 domain)
interface GigabitEthernet1/0/1
 no switchport
 ip address 10.49.0.33 255.255.255.252
interface GigabitEthernet1/0/2
 no switchport
 ip address 10.49.0.37 255.255.255.252
!
router ospf 1
 router-id 3.3.3.3
 network 10.49.0.4 0.0.0.3 area 0
 network 10.49.0.8 0.0.0.3 area 0
 redistribute rip subnets
!
! RIPv2 domain toward MS1 and MS2 (Oriente)
router rip
 version 2
 no auto-summary
 network 10.0.0.0
 redistribute ospf 1 metric 2
```

Como RIP declara redes con clase, `network 10.0.0.0` activa RIP en las interfaces del bloque 10.0.0.0/8. Las líneas `passive-interface Port-channel` de las plantillas históricas no forman parte de la configuración real verificada: esa sintaxis no fue aceptada por el IOS simulado del 3650. Los vecinos de enrutamiento del núcleo utilizan OSPF. El maestro añade en CORE-3 el agregado de retorno de Oriente de distancia 250/251 y `redistribute static subnets`, detallados en `Unido/CORE.conf`.

### 6.4 R-WAN

Fuente: [R-WAN.conf](R-WAN.conf). El archivo usa Serial0/3/0, correspondiente al HWIC-2T instalado en la ranura 3 del .pkt.

```
hostname R-WAN
!
! Link toward CORE-2 (OSPF domain)
interface GigabitEthernet0/0
 ip address 10.49.0.22 255.255.255.252
!
! Serial WAN toward R-Occidente (EIGRP domain, DCE side)
interface Serial0/3/0
 ip address 10.49.0.25 255.255.255.252
 clock rate 64000
!
router ospf 1
 router-id 4.4.4.4
 network 10.49.0.20 0.0.0.3 area 0
 redistribute eigrp 10 subnets
!
router eigrp 10
 network 10.49.0.24 0.0.0.3
 no auto-summary
  redistribute ospf 1 metric 1544 200 255 1 1500
```

### 6.5 R-CENTRAL1

Fuente: [R-CENTRAL1.conf](R-CENTRAL1.conf).

```
hostname R-CENTRAL1
!
! Link toward CORE-1 (OSPF domain)
interface GigabitEthernet0/0
 ip address 10.49.0.14 255.255.255.252
!
! Link toward R-CENTRAL2 (static segment)
interface GigabitEthernet0/1
 ip address 10.49.0.17 255.255.255.252
!
! Single summary route for the whole Data Center block
ip route 10.49.40.0 255.255.255.0 10.49.0.18
!
router ospf 1
 router-id 5.5.5.5
 network 10.49.0.12 0.0.0.3 area 0
 network 10.49.0.16 0.0.0.3 area 0
 passive-interface GigabitEthernet0/1
 redistribute static subnets
```

### 6.6 R-CENTRAL2

Fuente: [R-CENTRAL2.conf](R-CENTRAL2.conf).

```
hostname R-CENTRAL2
!
! Link toward R-CENTRAL1
interface GigabitEthernet0/0
 ip address 10.49.0.18 255.255.255.252
!
! Link toward MS-DC
interface GigabitEthernet0/1
 ip address 10.49.40.81 255.255.255.252
!
! Default route toward the rest of the network
ip route 0.0.0.0 0.0.0.0 10.49.0.17
!
! Static routes to each server VLAN through MS-DC
ip route 10.49.40.0 255.255.255.224 10.49.40.82
ip route 10.49.40.32 255.255.255.224 10.49.40.82
ip route 10.49.40.64 255.255.255.240 10.49.40.82
```

R-CENTRAL2 no corre ningún protocolo dinámico. Conoce el resto de la red solo por la ruta por defecto y conoce cada VLAN de servidores por una ruta estática explícita hacia MS-DC.

### 6.7 MS-DC

Fuente: [MS-DC.CONF](MS-DC.CONF).

```
hostname MS-DC
ip routing
!
vlan 15
 name Core_BD
vlan 25
 name Web_Apps
vlan 35
 name NOC
!
! Routed link toward R-CENTRAL2
interface GigabitEthernet1/0/1
 no switchport
 ip address 10.49.40.82 255.255.255.252
!
! Po1 toward SW-DC-ACC1
interface range GigabitEthernet1/0/2 - 3
 switchport mode trunk
 channel-group 1 mode active
interface Port-channel1
 switchport mode trunk
 switchport trunk allowed vlan 15,25,35
!
! Po2 toward SW-DC-ACC2
interface range GigabitEthernet1/0/4 - 5
 switchport mode trunk
 channel-group 2 mode active
interface Port-channel2
 switchport mode trunk
 switchport trunk allowed vlan 15,25,35
!
! Gateways for each server VLAN
interface Vlan15
 ip address 10.49.40.33 255.255.255.224
interface Vlan25
 ip address 10.49.40.1 255.255.255.224
interface Vlan35
 ip address 10.49.40.65 255.255.255.240
!
ip route 0.0.0.0 0.0.0.0 10.49.40.81
```

### 6.8 SW-DC-ACC1

Fuente: [SW_DC-ACC1.conf](SW_DC-ACC1.conf).

```
hostname SW-DC-ACC1
!
vlan 15
 name Core_BD
vlan 25
 name Web_Apps
!
! Po1 toward MS-DC
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 channel-group 1 mode active
interface Port-channel1
 switchport mode trunk
 switchport trunk allowed vlan 15,25,35
!
! BD1 to BD7
interface range FastEthernet0/1 - 7
 switchport mode access
 switchport access vlan 15
!
! WEB1 to WEB7
interface range FastEthernet0/8 - 14
 switchport mode access
 switchport access vlan 25
```

### 6.9 SW-DC-ACC2

Fuente: [SW-DC_ACC2.conf](SW-DC_ACC2.conf).

```
hostname SW-DC-ACC2
!
vlan 35
 name NOC
!
! Po2 toward MS-DC
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 channel-group 2 mode active
interface Port-channel2
 switchport mode trunk
 switchport trunk allowed vlan 15,25,35
!
! NOC1 to NOC7
interface range FastEthernet0/1 - 7
 switchport mode access
 switchport access vlan 35
```

## 7. Puntos de redistribución

| Equipo | Dominios | Qué propaga |
|---|---|---|
| R-WAN | OSPF 1 y EIGRP 10 | Redistribución mutua. Occidente aprende el resto de la red y el núcleo aprende las redes de Occidente |
| CORE-3 | OSPF 1 y RIPv2 | Redistribución mutua. Oriente aprende el resto de la red y el núcleo aprende las redes de Oriente |
| R-CENTRAL1 | Estáticas hacia OSPF 1 | Solo la ruta resumida 10.49.40.0/24 del Data Center, sin exponer el segmento estático |

Cada protocolo necesita un dato distinto para aceptar rutas que vienen de otro dominio.

En OSPF, `redistribute ... subnets` hace que se redistribuyan también las redes que no coinciden con su máscara de clase. Sin esa palabra, OSPF solo toma redes con clase completa, y como todas las redes del proyecto son subredes de 10.0.0.0/8 (por ejemplo 10.49.40.0/24), no se redistribuiría ninguna. Las rutas redistribuidas entran como externas de tipo 2 (O E2) con la métrica semilla por defecto de 20, que no aumenta al atravesar el núcleo. Por eso CORE-1 ve el Data Center como `O E2 10.49.40.0 [110/20]`.

En EIGRP, la configuración real de R-WAN usa `metric 1544 200 255 1 1500` como métrica semilla de las rutas que vienen de OSPF. Los cinco valores son ancho de banda en kbps (1544), retardo en decenas de microsegundos (200), confiabilidad (255), carga (1) y MTU (1500 bytes). El valor 2000 de la plantilla parcial no coincide con el maestro. EIGRP necesita una métrica semilla válida para instalar las rutas redistribuidas.

En RIP, `metric 2` asigna dos saltos a las rutas que vienen de OSPF. RIP tampoco puede traducir el costo de OSPF, y sin métrica explícita las rutas se anuncian como inalcanzables. Un valor bajo deja margen para que MS1 y MS2 las propaguen dentro de Oriente sin acercarse al límite de 15 saltos.

## 8. Verificaciones y pruebas

### 8.1 EtherChannel del núcleo

![Negociación LACP de Po12 en CORE-2](../Images/Screenshot%202026-10-18%20at%209.04.28%E2%80%AFPM.png)

La primera captura muestra la negociación LACP de Po12 en CORE-2. En el primer `show etherchannel summary`, Po12 aparece en (RD), de capa 3 pero caído, con Gi1/1/1 y Gi1/1/2 en (I), es decir, funcionando como puertos sueltos, porque el otro extremo todavía no respondía a LACP. En cuanto CORE-1 negocia, la consola registra que Port-channel12 cambia a up y que OSPF forma la adyacencia con 1.1.1.1 en FULL. En el segundo `show etherchannel summary`, Po12 ya está en (RU) con sus dos puertos en (P). La secuencia demuestra que el canal solo se forma cuando ambos extremos negocian LACP.

![show etherchannel summary y show ip ospf neighbor en CORE-2](../Images/Screenshot%202026-10-18%20at%209.04.58%E2%80%AFPM.png)

La segunda captura continúa la misma sesión. En CORE-2, `show etherchannel summary` muestra Po12 en (RU), es decir de capa 3 y en uso, con Gi1/1/1 y Gi1/1/2 en (P), agrupados en el canal por LACP. `show ip ospf neighbor` muestra a CORE-1 (1.1.1.1) en FULL sobre Port-channel12, lo que prueba que OSPF forma la adyacencia sobre la interfaz lógica y no sobre los puertos físicos. Po23 aparece todavía en (RD) porque la captura se tomó antes de configurar el extremo de CORE-3. Que Po23 quedó arriba después se demuestra en la sección 8.2, donde CORE-1 aprende 10.49.0.8/30 desde CORE-2 y desde CORE-3 al mismo tiempo, cosa que solo ocurre si ambos extremos de Po23 están activos.

### 8.2 Vecinos OSPF y tabla de rutas de CORE-1

![show ip ospf neighbor y show ip route ospf en CORE-1](../Images/Screenshot%202026-10-20%20at%2012.26.31%E2%80%AFPM.png)

`show ip ospf neighbor` en CORE-1 muestra a 2.2.2.2 sobre Port-channel12, a 3.3.3.3 sobre Port-channel13 y a 5.5.5.5 sobre Gi1/0/1, los tres en FULL, así que el triángulo y el enlace hacia la Sede Central están en OSPF. En `show ip route ospf`, la red 10.49.0.8/30 (Po23) aparece por dos caminos de costo 2, vía CORE-2 y vía CORE-3, lo que demuestra el balanceo de carga por costo igual (ECMP) que permite el núcleo de capa 3. La ruta `O E2 10.49.40.0 [110/20]` aprendida de R-CENTRAL1 prueba que la ruta estática del Data Center se redistribuye hacia OSPF como una sola ruta resumida. También aparece 10.49.0.16/30, el enlace pasivo hacia R-CENTRAL2, que se anuncia sin formar adyacencia.

### 8.3 EtherChannel y SVIs de MS-DC

![show etherchannel summary en MS-DC](../Images/Screenshot%202026-10-20%20at%2012.24.52%E2%80%AFPM.png)

En MS-DC, `show etherchannel summary` muestra Po1 (Gi1/0/2 y Gi1/0/3) y Po2 (Gi1/0/4 y Gi1/0/5) en (SU), es decir de capa 2 y en uso, con los cuatro puertos en (P) y LACP como protocolo. En los mensajes de la misma consola se ve que Vlan15, Vlan25 y Vlan35 cambian a up tanto en el estado de la interfaz como en el protocolo de línea, así que los tres gateways del Data Center están activos.

### 8.4 Conectividad dentro del Data Center

![Primera ejecución de las pruebas de PDU](../Images/Screenshot%202026-10-20%20at%2012.41.18%E2%80%AFPM.png)

La primera captura es la primera ejecución de las seis pruebas, la misma que aparece en el panel de la sección 3.2. Las pruebas dentro de cada VLAN y NOC1 a BD1 ya terminaron en Successful, mientras que BD1 a WEB1 y NOC7 a WEB7 aparecieron como Failed.

![Panel de PDUs con las seis pruebas exitosas](../Images/Screenshot%202026-10-20%20at%2012.42.01%E2%80%AFPM.png)

En la ejecución siguiente, a las 12:42, las seis pruebas de PDU terminaron en Successful. BD1 a BD7, WEB1 a WEB7 y NOC1 a NOC7 prueban la conectividad dentro de cada VLAN. BD1 a WEB1 prueba el enrutamiento inter-VLAN en MS-DC entre dos servidores conectados al mismo switch de acceso. NOC1 a BD1 y NOC7 a WEB7 prueban el enrutamiento inter-VLAN entre equipos de switches de acceso distintos, lo que además confirma que Po1 y Po2 transportan las tres VLANs.

## 9. Enlaces de integración con Oscar y Luis

Cuatro enlaces unen el backbone con los equipos de borde de las otras sedes. Mi extremo de cada uno está configurado y los cables se conectan al fusionar las partes en el .pkt maestro. Mientras no estén cableados, estas interfaces aparecen como down en mi archivo parcial, lo cual es esperado.

| Enlace | Extremo backbone (Mariano) | Extremo de la sede | Responsable | Red | Protocolo en ambos extremos |
|---|---|---|---|---|---|
| CORE-2 a R-Norte | CORE-2 Gi1/0/2, 10.49.0.29 | R-Norte Gi0/0, 10.49.0.30 | Oscar | 10.49.0.28/30 | OSPF 1, área 0 |
| R-WAN a R-Occidente | R-WAN Se0/3/0 (DCE, clock rate 64000), 10.49.0.25 | R-Occidente serial según su módulo (DTE), 10.49.0.26 | Oscar | 10.49.0.24/30 | EIGRP 10 |
| CORE-3 a MS1 | CORE-3 Gi1/0/1, 10.49.0.33 | MS1 Gi1/0/1 (puerto ruteado), 10.49.0.34 | Luis | 10.49.0.32/30 | RIPv2, sin auto-summary |
| CORE-3 a MS2 | CORE-3 Gi1/0/2, 10.49.0.37 | MS2 Gi1/0/1 (puerto ruteado), 10.49.0.38 | Luis | 10.49.0.36/30 | RIPv2, sin auto-summary |

Para que estos enlaces levanten sin cambios en el backbone, R-Norte debe anunciar sus redes y el enlace 10.49.0.28/30 en OSPF 1 área 0. R-Occidente necesita un módulo HWIC-2T (el router se apaga antes de insertarlo) y debe anunciar sus redes y el enlace 10.49.0.24/30 en EIGRP 10 con `no auto-summary`. MS1 y MS2 necesitan `ip routing`, el puerto Gi1/0/1 con `no switchport` y sus redes y su enlace anunciados en RIPv2 con `no auto-summary`.
