# Sección técnica — Sede Occidente y Sede Norte

**Proyecto:** BanTech GT — Redes de Computadoras 1 
**Grupo:** 4  



Este documento presenta la configuración y las pruebas realizadas en las sedes Occidente y Norte. Ambas topologías se reunieron en un mismo archivo de Packet Tracer. Por el momento, cada sede funciona de forma independiente; las conexiones con el backbone se realizarán durante la integración del proyecto general.
## 1. Objetivo y decisiones de diseño

La parte asignada consistió en implementar dos sedes con redes locales segmentadas por departamentos, capacidad de direccionamiento según VLSM y comunicación inter-VLAN a través de un router. Se adoptaron los parámetros del grupo `XX = 49`, `Y = 4`, dominio VTP `bantech49`, contraseña VTP `cisco` y bloque general de direccionamiento `10.49.0.0/16`.

**Occidente** representa la expansión regional. Se eligió una topología con un switch principal, cuatro switches de acceso y un router Cisco 2911. Las VLANs separan Cajas, Asesores, Gerencia y Seguridad, reduciendo los dominios de broadcast y facilitando la administración. El router realiza *Router-on-a-Stick* mediante subinterfaces 802.1Q y tiene configurado **EIGRP AS 10** para anunciar sus redes al borde WAN cuando se complete el enlace con R-WAN.

**Norte** corresponde a una sede crítica de autorización. Se eligió un switch principal y tres switches de acceso enlazados entre sí con caminos redundantes. Se configuró **Rapid PVST+** para evitar bucles y conservar rutas alternativas ante una falla. Se eligió `SW-N-PRINCIPAL` como **Root Bridge** de las VLANs 54, 64 y 74 mediante prioridad reducida, porque conecta directamente con R-Norte y permite mantener una ruta lógica principal predecible hacia las puertas de enlace. R-Norte utiliza **OSPF proceso 1, área 0**, conforme a la conexión prevista con CORE-2.

## 2. Dispositivos y topología

| Sede | Dispositivos de red | Equipos finales | Diseño |
|---|---|---:|---|
| Occidente | `R-Occidente` (2911), `SW-O-PRINCIPAL` y cuatro switches de acceso 2960 | 28 PCs: 7 por departamento | Estrella con troncales hacia el switch principal |
| Norte | `R-Norte` (2911), `SW-N-PRINCIPAL` y tres switches de acceso 2960 | 21 PCs: 7 por departamento | Switch principal y enlaces redundantes entre los switches de acceso |

**Total de la parte entregada:** 49 PCs, 7 VLANs de usuarios, 2 routers y 9 switches. El archivo conjunto contiene ambas sedes, sin enlace físico entre sus routers.

**Evidencias de topología y cableado:**

 `01_Topologia_Inicial_Occidente.jpeg`, 
![alt text](capturas/01_Topologia_Inicial_Occidente.jpeg)

`02_Conexiones_Fisicas_Occidente.jpeg`. 
![alt text](capturas/02_Conexiones_Fisicas_Occidente.jpeg)
## 3. Plan de direccionamiento VLSM

Las subredes se eligieron para cubrir la capacidad de hosts solicitada para cada departamento (no solo las siete PCs representadas). En cada VLAN se reservó una dirección para la puerta de enlace del router.

### 3.1 Sede Occidente — bloque `10.49.10.0/24`

| VLAN | Departamento | Hosts requeridos | Red / prefijo | Máscara | Gateway | Hosts utilizables de la subred |
|---|---|---:|---|---|---|---|
| 14 | CAJAS | 45 | `10.49.10.0/26` | `255.255.255.192` | `10.49.10.1` | `10.49.10.1–10.49.10.62` |
| 24 | ASESORES | 30 | `10.49.10.64/26` | `255.255.255.192` | `10.49.10.65` | `10.49.10.65–10.49.10.126` |
| 34 | GERENCIA | 10 | `10.49.10.144/28` | `255.255.255.240` | `10.49.10.145` | `10.49.10.145–10.49.10.158` |
| 44 | SEGURIDAD | 12 | `10.49.10.128/28` | `255.255.255.240` | `10.49.10.129` | `10.49.10.129–10.49.10.142` |

**Direcciones de PCs representadas:** Cajas `.2–.8`; Asesores `.66–.72`; Gerencia `.146–.152`; Seguridad `.130–.136` (todas bajo `10.49.10.` y la máscara de su VLAN). Los rangos indicados en la tabla incluyen la IP del gateway, por lo que no todas están libres para PCs.

### 3.2 Sede Norte — bloque `10.49.20.0/24`

| VLAN | Departamento | Hosts requeridos | Red / prefijo | Máscara | Gateway | Hosts utilizables de la subred |
|---|---|---:|---|---|---|---|
| 54 | ANALISIS | 50 | `10.49.20.0/26` | `255.255.255.192` | `10.49.20.1` | `10.49.20.1–10.49.20.62` |
| 64 | AUDITORIA | 25 | `10.49.20.64/27` | `255.255.255.224` | `10.49.20.65` | `10.49.20.65–10.49.20.94` |
| 74 | LEGAL | 14 | `10.49.20.96/27` | `255.255.255.224` | `10.49.20.97` | `10.49.20.97–10.49.20.126` |

**Nota de dimensionamiento:** para Legal se reservó `/27`, porque se necesitan 14 equipos más una dirección de gateway; una `/28` permitiría solo 14 direcciones utilizables en total.

**Direcciones de PCs representadas:** Análisis `10.49.20.2–.8` (`PC-A1` a `PC-A7`); Auditoría `10.49.20.66–.72` (`PC-U1` a `PC-U7`); Legal `10.49.20.98–.104` (`PC-L1` a `PC-L7`).

### 3.3 Enlaces previstos hacia el backbone

| Enlace | Extremo de Oscar | Extremo del backbone | Red | Protocolo |
|---|---|---|---|---|
| Occidente → R-WAN | `R-Occidente Se0/3/0`: `10.49.0.26/30` | `R-WAN Se0/3/0`: `10.49.0.25/30` (DCE; `clock rate 64000`) | `10.49.0.24/30` | EIGRP 10 |
| Norte → CORE-2 | `R-Norte Gi0/0`: `10.49.0.30/30` | `CORE-2 Gi1/0/2`: `10.49.0.29/30` | `10.49.0.28/30` | OSPF 1, área 0 |

**Estado:** interfaces configuradas en las sedes, con enlace físico aún pendiente de conectar en el archivo maestro. `down/down` y ausencia de vecinos en el archivo parcial son condiciones esperadas antes del cableado.

## 4. VLANs, VTP, troncales y acceso

Se configuró VTP versión 2, dominio `bantech49` y contraseña `cisco`. En cada sede hay un switch principal en modo **server** y switches de acceso en modo **client**. Las VLANs de las sedes se administran como dominios de capa 2 separados; **no deben unirse sus switches mediante un troncal de capa 2**, ya que comparten nombre de dominio VTP pero poseen bases de VLAN independientes. El enlace al backbone se hace mediante equipos de capa 3.

En Occidente, el switch principal y sus cuatro switches de acceso transportan las VLANs 14, 24, 34 y 44. Cada grupo de siete PCs está conectado a puertos de acceso de su propia VLAN. El enlace hacia `R-Occidente Gi0/0` funciona como troncal 802.1Q.

En Norte, `SW-N-PRINCIPAL` enlaza `Fa0/1` con Análisis, `Fa0/2` con Auditoría, `Fa0/3` con Legal, y `Gi0/1` con `R-Norte Gi0/1`. Las cuatro interfaces operan como troncales 802.1Q y permiten exclusivamente las VLANs `54,64,74`. En los switches de acceso, `Fa0/1–Fa0/7` se destinan a las siete PCs de cada departamento.

**Capturas de Occidente:** 

`03_VTP_Server_Occidente.jpeg`,
![alt text](capturas/03_VTP_Server_Occidente.jpeg)
 `04_VLANs_Server_Occidente.jpeg`, 
![alt text](capturas/04_VLANs_Server_Occidente.jpeg)
`05_Trunks_Principal_Occidente.jpeg`, 
 ![alt text](capturas/05_Trunks_Principal_Occidente.jpeg)`06_VTP_Client_Cajas_Occidente.jpeg`,
 ![alt text](capturas/06_VTP_Client_Cajas_Occidente.jpeg)
`07_Puertos_Access_Cajas_Occidente.jpeg`,
![alt text](capturas/07_Puertos_Access_Cajas_Occidente.jpeg)
`08_VTP_Client_Asesores_Occidente.jpeg`, 
![alt text](capturas/08_VTP_Client_Asesores_Occidente.jpeg)
`09_VLAN24_Puertos_Asesores.jpeg`,
![alt text](capturas/09_VLAN24_Puertos_Asesores.jpeg)
`10_VTP_Client_Gerencia_Occidente.jpeg`, 
![alt text](capturas/10_VTP_Client_Gerencia_Occidente.jpeg)
`11_VLAN34_Puertos_Gerencia.jpeg`,
![alt text](capturas/11_VLAN34_Puertos_Gerencia.jpeg)
`12_VTP_Client_Seguridad_Occidente.jpeg`, 
![alt text](capturas/12_VTP_Client_Seguridad_Occidente.jpeg)
`13_VLAN44_Puertos_Seguridad.jpeg`.
![alt text](capturas/13_VLAN44_Puertos_Seguridad.jpeg)

**Capturas de Norte:** 

`35_Trunks_SW_N_Principal.jpeg`
![alt text](capturas/35_Trunks_SW_N_Principal.jpeg)
`42_Puertos_Acceso_VLAN54_Analisis.jpeg`
![alt text](capturas/42_Puertos_Acceso_VLAN54_Analisis.jpeg)
`43_Puertos_Acceso_VLAN64_Auditoria.jpeg`
![alt text](capturas/43_Puertos_Acceso_VLAN64_Auditoria.jpeg)
`44_Puertos_Acceso_VLAN74_Legal.jpeg`.
![alt text](capturas/44_Puertos_Acceso_VLAN74_Legal.jpeg)


## 5. Enrutamiento inter-VLAN y protocolos hacia el backbone

### 5.1 R-Occidente — Router-on-a-Stick y EIGRP

Extracto representativo de los parámetros comprobados en `show ip interface brief` y `show ip protocols`:

```text
R-Occidente — subinterfaces e IP
Gi0/0.14    10.49.10.1/26   VLAN 14 CAJAS
Gi0/0.24    10.49.10.65/26  VLAN 24 ASESORES
Gi0/0.34    10.49.10.145/28 VLAN 34 GERENCIA
Gi0/0.44    10.49.10.129/28 VLAN 44 SEGURIDAD
Se0/3/0     10.49.0.26/30  hacia R-WAN (aún sin cable)

EIGRP: AS 10, Router ID 10.49.0.26, no auto-summary
Redes: 10.49.0.24/30, 10.49.10.0/26, 10.49.10.64/26,
       10.49.10.128/28, 10.49.10.144/28
Interfaces pasivas: Gi0/0.14, Gi0/0.24, Gi0/0.34, Gi0/0.44
```

Las subinterfaces físicas se configuran con encapsulación `dot1Q` del ID de VLAN correspondiente; la interfaz física sirve como transporte del troncal y no necesita dirección IP propia. Las interfaces pasivas evitan enviar actualizaciones EIGRP a las VLANs de usuarios.

**Capturas:**

`16_Subinterfaces_ROAS_Occidente.jpeg`
![alt text](capturas/16_Subinterfaces_ROAS_Occidente.jpeg)
`29_Instalacion_HWIC2T_Occidente.jpeg`
![alt text](capturas/29_Instalacion_HWIC2T_Occidente.jpeg)
`31_Interfaz_Serial_R_Occidente.jpeg`
![alt text](capturas/31_Interfaz_Serial_R_Occidente.jpeg)
`32_EIGRP_Protocolos_Occidente.jpeg`
![alt text](capturas/32_EIGRP_Protocolos_Occidente.jpeg)
`33_Tabla_Rutas_Occidente.jpeg`
![alt text](capturas/33_Tabla_Rutas_Occidente.jpeg)
`34_Vecinos_EIGRP_Occidente.jpeg`.
![alt text](capturas/34_Vecinos_EIGRP_Occidente.jpeg)


### 5.2 R-Norte — Router-on-a-Stick y OSPF

```text
R-Norte — subinterfaces e IP
Gi0/1.54    10.49.20.1/26   VLAN 54 ANALISIS
Gi0/1.64    10.49.20.65/27  VLAN 64 AUDITORIA
Gi0/1.74    10.49.20.97/27  VLAN 74 LEGAL
Gi0/0       10.49.0.30/30  hacia CORE-2 (aún sin cable)

OSPF: proceso 1, Router ID 10.49.0.30, área 0
network 10.49.0.28 0.0.0.3 area 0
network 10.49.20.0 0.0.0.63 area 0
network 10.49.20.64 0.0.0.31 area 0
network 10.49.20.96 0.0.0.31 area 0
passive-interface GigabitEthernet0/1.54
passive-interface GigabitEthernet0/1.64
passive-interface GigabitEthernet0/1.74
```

El enlace hacia el backbone usa OSPF en área 0, mientras que las subinterfaces de usuarios son pasivas y solo anuncian sus redes. Se verificaron las redes con `show ip protocols`; sin conexión a CORE-2 no se espera una adyacencia OSPF externa.

**Capturas:** 
`57_OSPF_Protocolos_R_Norte.jpeg`

![alt text](capturas/57_OSPF_Protocolos_R_Norte.jpeg)
`58_Tabla_Rutas_R_Norte.jpeg`

![alt text](capturas/58_Tabla_Rutas_R_Norte.jpeg)
`59_Vecinos_OSPF_R_Norte.jpeg`.

![alt text](capturas/59_Vecinos_OSPF_R_Norte.jpeg)

## 6. Rapid PVST+ y elección del Root Bridge en Norte

En los cuatro switches de Norte se habilitó Rapid PVST+ (`spanning-tree mode rapid-pvst`). El switch `SW-N-PRINCIPAL` se configuró como raíz de las VLANs 54, 64 y 74 (`spanning-tree vlan 54,64,74 root primary`). Esta decisión coloca la raíz en el punto de concentración de los troncales y del enlace hacia el router, y deja que los switches de acceso mantengan caminos alternativos sin crear bucles.

La topología redundante conecta los switches de acceso entre sí además de sus enlaces directos con el principal. Rapid PVST+ pone algunos puertos en estado alternativo/bloqueado (`Altn BLK`) para impedir la circulación indefinida de tramas. **Los enlaces naranjas por bloqueo STP son esperados**, no deben forzarse a verde.

Después de combinar las sedes, `show spanning-tree vlan 54` confirmó en `SW-N-PRINCIPAL`: protocolo `rstp`, prioridad de raíz `24630`, dirección `0001.C94A.3EA7` y mensaje `This bridge is the root`; todos sus enlaces troncales relevados se encontraban en `Desg FWD`. En `SW-N-ANALISIS`, `Fa0/24` permaneció `Root FWD`, con un puerto alternativo bloqueado (la distribución de los puertos alternativos puede variar cuando se copian los dispositivos).

**Evidencias de elección de raíz y enlaces:** 

`36_Root_Bridge_VLAN54_Norte.jpeg`,
![alt text](capturas/36_Root_Bridge_VLAN54_Norte.jpeg)

`37_Root_Bridge_VLAN64_Norte.jpeg`,
![alt text](capturas/37_Root_Bridge_VLAN64_Norte.jpeg)

`38_Root_Bridge_VLAN74_Norte.jpeg`,
![alt text](capturas/38_Root_Bridge_VLAN74_Norte.jpeg)

`39_RapidPVST_Puerto_Bloqueado_Analisis.jpeg`, 
![alt text](capturas/39_RapidPVST_Puerto_Bloqueado_Analisis.jpeg)

`41_RapidPVST_Estado_Puertos_Legal.jpeg`.
![alt text](capturas/41_RapidPVST_Estado_Puertos_Legal.jpeg)

### 6.1 Prueba de falla y recuperación del enlace (archivo parcial de Norte)

Se ensayó el apagado temporal del enlace principal de `SW-N-ANALISIS` por `Fa0/24` (`shutdown`). Tras la reconvergencia, STP seleccionó el camino alternativo por `Fa0/22` como `Root FWD`, con costo de ruta de 38 frente al costo 19 del enlace directo. Las pruebas de ping desde Análisis hacia su gateway y hacia Legal respondieron tras la convergencia. Después se restauró `Fa0/24` con `no shutdown`: el enlace directo volvió a `Root FWD` y el camino alternativo regresó al estado bloqueado.

**Alcance preciso:** estas capturas confirman comunicación después de converger y restauración del estado STP. **No representan una medición del tiempo de convergencia en milisegundos** ni sustituyen la repetición de la prueba sobre una copia del `.pkt` maestro integrado.

**Capturas:**

 `60_RapidPVST_Recuperacion_Enlace_Norte.jpeg`, 
 ![alt text](capturas/60_RapidPVST_Recuperacion_Enlace_Norte.jpeg)

 `61_Ping_Recuperacion_RapidPVST_Norte.jpeg`, 
 ![alt text](capturas/61_Ping_Recuperacion_RapidPVST_Norte.jpeg)

 `62_RapidPVST_Enlace_Restaurado_Norte.jpeg`.
![alt text](capturas/62_RapidPVST_Enlace_Restaurado_Norte.jpeg)

## 7. Verificaciones de conectividad

### 7.1 Sede Occidente

Se verificaron direcciones IP de PCs de cada departamento y conectividad entre VLANs mediante `ping`. Las pruebas representativas desde Cajas incluyeron el gateway `10.49.10.1`, un equipo de Asesores `10.49.10.66`, uno de Gerencia `10.49.10.146` y uno de Seguridad `10.49.10.130`.

**Capturas:** 

`17_IP_PC0_Cajas.jpeg`,
![alt text](capturas/17_IP_PC0_Cajas.jpeg)

`18_Ping_PC0_Gateway_Cajas.jpeg`, 
![alt text](capturas/18_Ping_PC0_Gateway_Cajas.jpeg)

`19_IP_PC7_Asesores.jpeg`, 
![alt text](capturas/19_IP_PC7_Asesores.jpeg)

`20_Ping_InterVLAN_Cajas_Asesores.jpeg`, 
![alt text](capturas/20_Ping_InterVLAN_Cajas_Asesores.jpeg)

`21_IP_PC14_Gerencia.jpeg`, 
![alt text](capturas/21_IP_PC14_Gerencia.jpeg)

`22_Ping_InterVLAN_Gerencia.jpeg`, 
![alt text](capturas/22_Ping_InterVLAN_Gerencia.jpeg)

`23_IP_PC21_Seguridad.jpeg`, 
![alt text](capturas/23_IP_PC21_Seguridad.jpeg)

`24_Ping_InterVLAN_Seguridad.jpeg`, 
![alt text](capturas/24_Ping_InterVLAN_Seguridad.jpeg)

`25_Ping_PC6_Gateway_Cajas.jpeg`, 
![alt text](capturas/25_Ping_PC6_Gateway_Cajas.jpeg)

`26_Ping_PC13_Gateway_Asesores.jpeg`, 
![alt text](capturas/26_Ping_PC13_Gateway_Asesores.jpeg)

`27_Ping_PC20_Gateway_Gerencia.jpeg`, 
![alt text](capturas/27_Ping_PC20_Gateway_Gerencia.jpeg)

`28_Ping_PC27_Gateway_Seguridad.jpeg`.
![alt text](capturas/28_Ping_PC27_Gateway_Seguridad.jpeg)

### 7.2 Sede Norte

Se verificaron las configuraciones IP y los gateways de las tres áreas y los pings inter-VLAN Análisis–Auditoría, Análisis–Legal y Auditoría–Legal. Las pruebas representativas desde `PC-A1` incluyeron `10.49.20.1` (gateway), `10.49.20.66` (Auditoría) y `10.49.20.98` (Legal).

**Capturas:** 

`45_IP_PC_A1_Analisis.jpeg`,
![alt text](capturas/45_IP_PC_A1_Analisis.jpeg)
`46_IP_PC_U1_Auditoria.jpeg`, 
![alt text](capturas/46_IP_PC_U1_Auditoria.jpeg)
`47_IP_PC_L1_Legal.jpeg`, 
![alt text](capturas/47_IP_PC_L1_Legal.jpeg)
`48_Ping_PC_A1_Gateway_Analisis.jpeg`,
![alt text](capturas/48_Ping_PC_A1_Gateway_Analisis.jpeg)
`49_Ping_PC_U1_Gateway_Auditoria.jpeg`,
![alt text](capturas/49_Ping_PC_U1_Gateway_Auditoria.jpeg)
`50_Ping_PC_L1_Gateway_Legal.jpeg`, 
![alt text](capturas/50_Ping_PC_L1_Gateway_Legal.jpeg)
`51_Ping_InterVLAN_Analisis_Auditoria.jpeg`, 
![alt text](capturas/51_Ping_InterVLAN_Analisis_Auditoria.jpeg)
`52_Ping_InterVLAN_Analisis_Legal.jpeg`, 
![alt text](capturas/52_Ping_InterVLAN_Analisis_Legal.jpeg)
`53_Ping_InterVLAN_Auditoria_Legal.jpeg`, 
![alt text](capturas/53_Ping_InterVLAN_Auditoria_Legal.jpeg)
`54_Ping_PC_A7_Gateway_Analisis.jpeg`, 
![alt text](capturas/54_Ping_PC_A7_Gateway_Analisis.jpeg)
`55_Ping_PC_U7_Gateway_Auditoria.jpeg`, 
![alt text](capturas/55_Ping_PC_U7_Gateway_Auditoria.jpeg)
`56_Ping_PC_L7_Gateway_Legal.jpeg`.
![alt text](capturas/56_Ping_PC_L7_Gateway_Legal.jpeg)


### 7.3 Revisión posterior a la unión de los dos archivos parciales

Se creó `BanTechGT_Sedes_Oscar_Grupo4.pkt`, se cerró y volvió a abrir, y se repitieron pings representativos: Occidente `PC0 → 10.49.10.66` y Norte `PC-A1 → 10.49.20.98`. En ambos casos el segundo intento respondió **4/4 con 0 % de pérdida**. Los primeros intentos perdieron uno o dos paquetes, comportamiento compatible con inicialización y ARP de Packet Tracer. Esto verifica comunicación interna de cada sede, **no conectividad entre Occidente y Norte ni con el Data Center**.


## 8. Integración con el proyecto general

El archivo `BanTechGT_Sedes_Oscar_Grupo4.pkt` reúne las dos sedes y se encuentra en la carpeta `Oscar` de la rama `Oscar-Sedes-Occidente-Norte`. La configuración local y la comunicación entre VLANs se comprobaron antes y después de guardar el archivo. La integración con el backbone y las pruebas entre sedes corresponden a la siguiente etapa del trabajo en equipo.

### 8.1 Conexiones previstas

| Conexión | Puerto de nuestra sede | Puerto del backbone | Direccionamiento |
|---|---|---|---|
| Occidente – R-WAN | `R-Occidente Se0/3/0` | `R-WAN Se0/3/0` (DCE) | `10.49.0.26/30` ↔ `10.49.0.25/30` |
| Norte – CORE-2 | `R-Norte Gi0/0` | `CORE-2 Gi1/0/2` | `10.49.0.30/30` ↔ `10.49.0.29/30` |

### 8.2 Pruebas para la etapa de integración

Una vez unidas las topologías al backbone, se comprobarán las vecindades EIGRP y OSPF, las rutas aprendidas y los pings desde Occidente y Norte hacia otras sedes y el Data Center. También se repetirá la prueba de recuperación de enlaces de Norte sobre una copia del proyecto integrado, tomando nuevas capturas para el manual general.

Los resultados de esas pruebas se agregarán al documento cuando se realicen. Las pruebas locales ya documentadas en las secciones anteriores no necesitan presentarse como pendientes.
## 9. Índice de evidencias y organización sugerida

El ZIP de evidencias entregado contiene capturas numeradas de `01` a `62` (63 imágenes, porque existen dos archivos diferentes con prefijo `14_`). **Las capturas no vienen embebidas en este `.md` descargable**. Para que Luis pueda abrirlas desde el repositorio, conviene copiar los archivos de `capturas/` a una carpeta bajo `Oscar/capturas/` y conservar los nombres originales; luego vincular o incrustar en el README consolidado únicamente las imágenes que correspondan.

| Bloque | Capturas | Comandos y comprobaciones |
|---|---|---|
| Occidente: topología y VLANs | 01–13 | Diseño físico, VTP server/client, `show vlan brief`, `show interfaces trunk` |
| Occidente: ROAS y pings | 14–28 | Subinterfaces del router, IP y `ping` por VLAN |
| Occidente: WAN y EIGRP | 29–34 | HWIC-2T, serial, `show ip protocols`, `show ip route`, vecinos |
| Norte: VLANs y Rapid PVST+ | 35–44 | Troncales, `show spanning-tree vlan 54/64/74`, puertos de acceso |
| Norte: IP y pings | 45–56 | Configuración IP, gateways y pings inter-VLAN |
| Norte: OSPF y contingencia | 57–62 | `show ip protocols`, rutas, vecinos y recuperación STP |

**Fuentes de trabajo:** `Oscar/TareasOscar.md`, `README.md` de planificación del repositorio, documentación de backbone de Mariano (`Mariano/README.md`), salidas de CLI verificadas y capturas proporcionadas en `evidencias de proyecto2 redes.zip`.
