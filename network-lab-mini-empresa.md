# Network Lab — Mini Empresa
### TechCorp S.A. | Cisco Packet Tracer

---

## Resumen del Proyecto

Este laboratorio simula la red completa de una empresa pequeña (TechCorp S.A., 15 empleados) usando Cisco Packet Tracer. Cubre los conceptos fundamentales de redes que se usan en cualquier empresa: segmentación por VLANs, asignación automática de IPs con DHCP, resolución de nombres con DNS, servidor web interno y control de acceso con ACLs.

**Tecnologías utilizadas:**
- Cisco Packet Tracer — simulador de redes
- ISR 4331 — router principal con soporte para subinterfaces y DHCP
- Catalyst 3560 — switch multicapa (capa 3)
- Catalyst 2960 — switch de acceso (capa 2)
- VLANs 802.1Q — segmentación lógica de la red
- DNS y HTTP — servicios de red internos
- ACLs extendidas — control de acceso entre VLANs

---

## Topología de Red

La red sigue una arquitectura jerárquica de 3 capas, que es el estándar en redes empresariales:

```
Internet / Cloud
      |
   Router0 (ISR 4331)         <- Gateway, NAT, DHCP, ACLs
      |
   Multilayer Switch2 (3560)  <- Capa de distribución
      |
   Switch0 (2960)             <- Capa de acceso
   /    |    |    |    \
 PC0   PC1  ...  Server0  Server1
```

> **Por qué esta arquitectura:** Separa funciones claramente. El router maneja la conexión a internet y las políticas de seguridad. El switch multicapa gestiona el routing interno entre VLANs. El switch de acceso conecta los dispositivos finales. Esto hace la red escalable y fácil de troubleshootear.

> 💼 **En trabajo real:** Cuando llegas a una empresa a hacer soporte o auditoría, lo primero que pides es el diagrama de red. Si la empresa no tiene uno, crearlo es una de las primeras tareas. Saber leer y explicar esta arquitectura es pregunta común en entrevistas de IT support y network junior.

---

## Plan de Direccionamiento IP

Antes de configurar cualquier dispositivo, se define el plan de IPs. Esta es documentación crítica en cualquier empresa.

| VLAN | Nombre | Subred | Gateway | Rango DHCP | Propósito |
|------|--------|--------|---------|------------|-----------|
| VLAN 10 | Administración | 192.168.10.0/24 | 192.168.10.1 | .100 – .150 | PCs de administración |
| VLAN 20 | Ventas | 192.168.20.0/24 | 192.168.20.1 | .100 – .150 | PCs de ventas |
| VLAN 30 | IT | 192.168.30.0/24 | 192.168.30.1 | .100 – .150 | Equipo de IT |
| VLAN 40 | Servidores | 192.168.40.0/24 | 192.168.40.1 | IPs estáticas | DNS y Web server |
| VLAN 99 | Management | 192.168.99.0/24 | 192.168.99.1 | Solo IT | Administrar dispositivos |

**Servidores con IP estática:**

| Dispositivo | IP | Rol |
|-------------|----|-----|
| Server1 (DNS) | 192.168.40.10 | Resuelve nombres internos a IPs |
| Server0 (Web) | 192.168.40.20 | Sirve la intranet de TechCorp |

> **Por qué los servidores tienen IP estática:** Si un servidor usara DHCP podría cambiar de IP en cualquier momento. El DNS dejaría de funcionar, las ACLs apuntarían a la IP incorrecta y nadie podría conectarse. Regla de oro: **servidores = IP estática, siempre documentada.**

---

## VLANs — Segmentación de Red

### ¿Qué es una VLAN?

Una VLAN (Virtual LAN) divide un switch físico en múltiples redes lógicas separadas. Los dispositivos en VLAN 10 no pueden hablar con VLAN 20 a menos que el router lo permita explícitamente. Es como tener múltiples switches físicos separados, pero en un solo dispositivo.

> 💼 **En trabajo real:** En cualquier empresa mediana o grande, todo está segmentado por VLANs. Finanzas no puede ver la red de IT, los servidores están aislados, los invitados tienen su propia VLAN. Si un malware entra a la red de ventas, las VLANs evitan que se propague a los servidores. Configurar y troubleshootear VLANs es tarea diaria en IT support y network admin.

### Crear VLANs en el Switch

```
Switch> enable
Switch# configure terminal

Switch(config)# vlan 10
Switch(config-vlan)# name ADMINISTRACION
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name VENTAS
Switch(config-vlan)# exit

Switch(config)# vlan 30
Switch(config-vlan)# name IT
Switch(config-vlan)# exit

Switch(config)# vlan 40
Switch(config-vlan)# name SERVIDORES
Switch(config-vlan)# exit

Switch(config)# vlan 99
Switch(config-vlan)# name MANAGEMENT
Switch(config-vlan)# exit

! Verificar que quedaron creadas
Switch# show vlan brief
```

### Asignar Puertos de Acceso a cada VLAN

Un puerto de acceso pertenece a una sola VLAN. Se usa para conectar dispositivos finales: PCs, impresoras, servidores.

```
! Puertos 1-4 para VLAN 10 (Administración)
Switch(config)# interface range FastEthernet0/1-4
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# exit

! Puertos 5-8 para VLAN 20 (Ventas)
Switch(config)# interface range FastEthernet0/5-8
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
Switch(config-if-range)# exit

! Puertos 9-12 para VLAN 30 (IT)
Switch(config)# interface range FastEthernet0/9-12
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 30
Switch(config-if-range)# exit

! Puertos 13-14 para VLAN 40 (Servidores)
Switch(config)# interface range FastEthernet0/13-14
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 40
Switch(config-if-range)# exit
```

### Puerto Trunk hacia el Router (802.1Q)

Un puerto trunk lleva tráfico de MÚLTIPLES VLANs etiquetado con 802.1Q. Es como una autopista que lleva tráfico de todas las VLANs al router.

```
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30,40,99
Switch(config-if)# exit

! Verificar el trunk
Switch# show interfaces trunk
```

> **Access vs Trunk:**
> - Puerto **access** → conecta UN dispositivo final a UNA VLAN. El dispositivo no sabe que existe la VLAN.
> - Puerto **trunk** → conecta switches entre sí o switch con router. Lleva MÚLTIPLES VLANs con etiquetas 802.1Q.

> ⚠️ **Error clásico:** Configurar el puerto que va al router como `access` en vez de `trunk`. Si el puerto no es trunk, el router solo ve una VLAN y el inter-VLAN routing no funciona. Siempre verificar con `show interfaces trunk`.

---

## Router — Subinterfaces y DHCP

### Router-on-a-Stick

Con un solo cable físico entre el router y el switch, creamos múltiples subinterfaces virtuales — una por cada VLAN. Cada subinterfaz tiene su propia IP que actúa como gateway de esa VLAN.

> 💼 **En trabajo real:** En empresas pequeñas y medianas es común tener un solo router manejando múltiples VLANs con esta técnica. En empresas grandes se usan switches de capa 3 para el routing interno. Si en una entrevista te preguntan cómo hacer inter-VLAN routing con un solo router, esta es la respuesta.

### Configurar Subinterfaces

```
Router> enable
Router# configure terminal

! Subinterfaz para VLAN 10
Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit

! Subinterfaz para VLAN 20
Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit

! Subinterfaz para VLAN 30
Router(config)# interface GigabitEthernet0/0.30
Router(config-subif)# encapsulation dot1Q 30
Router(config-subif)# ip address 192.168.30.1 255.255.255.0
Router(config-subif)# exit

! Subinterfaz para VLAN 40
Router(config)# interface GigabitEthernet0/0.40
Router(config-subif)# encapsulation dot1Q 40
Router(config-subif)# ip address 192.168.40.1 255.255.255.0
Router(config-subif)# exit

! Activar la interfaz física principal (sin esto nada funciona)
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no shutdown
Router(config-if)# exit
```

> **Importante:** `encapsulation dot1Q 10` le dice al router que esta subinterfaz maneja tráfico etiquetado con VLAN ID 10. El número al final **DEBE** coincidir con el número de la VLAN.

### Configurar DHCP por VLAN

DHCP (Dynamic Host Configuration Protocol) asigna IPs automáticamente a los dispositivos cuando se conectan a la red.

> 💼 **En trabajo real:** DHCP es uno de los servicios más críticos de cualquier red empresarial. Si el servidor DHCP cae, los dispositivos no obtienen IP y nadie tiene red. En IT support recibirás tickets de "no tengo internet" que en realidad son problemas de DHCP.

```
! Excluir IPs que no queremos que DHCP asigne
! (gateways, servidores, dispositivos con IP fija)
Router(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.99
Router(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.99
Router(config)# ip dhcp excluded-address 192.168.30.1 192.168.30.99
Router(config)# ip dhcp excluded-address 192.168.40.1 192.168.40.99

! Pool DHCP para VLAN 10
Router(config)# ip dhcp pool VLAN10-ADMIN
Router(dhcp-config)# network 192.168.10.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.10.1
Router(dhcp-config)# dns-server 192.168.40.10
Router(dhcp-config)# exit

! Pool DHCP para VLAN 20
Router(config)# ip dhcp pool VLAN20-VENTAS
Router(dhcp-config)# network 192.168.20.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.20.1
Router(dhcp-config)# dns-server 192.168.40.10
Router(dhcp-config)# exit

! Pool DHCP para VLAN 30
Router(config)# ip dhcp pool VLAN30-IT
Router(dhcp-config)# network 192.168.30.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.30.1
Router(dhcp-config)# dns-server 192.168.40.10
Router(dhcp-config)# exit

! Comandos de verificación
Router# show ip dhcp binding    ! Ver IPs asignadas
Router# show ip dhcp pool       ! Ver estado de los pools
Router# show ip dhcp conflict   ! Ver conflictos de IP
```

> ⚠️ **Error clásico:** Si un PC no recibe IP por DHCP casi siempre es uno de estos: (1) el puerto del switch no está en la VLAN correcta, (2) olvidaste hacer `no shutdown` en la interfaz del router, (3) el pool DHCP no tiene IPs disponibles. Verificar con `show ip dhcp binding`.

---

## DNS y Servidores Internos

### ¿Qué es DNS?

DNS (Domain Name System) traduce nombres legibles por humanos (`intranet.techcorp.local`) a direcciones IP (`192.168.40.20`). Sin DNS tendrías que memorizar IPs para acceder a cualquier servicio.

> 💼 **En trabajo real:** Todas las empresas tienen DNS interno. Cuando un empleado escribe `sharepoint.empresa.com` o accede a la intranet, hay un servidor DNS interno resolviendo esos nombres a IPs privadas. El comando más útil: `nslookup nombre.dominio` para verificar que el DNS resuelve correctamente.

### Server1 — Servidor DNS (192.168.40.10)

En Packet Tracer: click en Server1 → Services → DNS → **ON**

| Nombre (Name) | Tipo | IP (Address) |
|---------------|------|--------------|
| intranet.techcorp.local | A Record | 192.168.40.20 |
| dns.techcorp.local | A Record | 192.168.40.10 |

> **Por qué `.local`:** Es la convención para redes internas — nunca se registra en internet. Separa claramente lo interno de lo externo. En empresa real verás `servidor.empresa.local` o `dc01.corp.local`. El tipo **A Record** mapea un nombre a una IPv4.

### Server0 — Servidor Web (192.168.40.20)

En Packet Tracer: click en Server0 → Services → HTTP → **ON**

Editar el archivo `index.html` con el contenido de la intranet de TechCorp.

> **Por qué servidores separados:** Si un atacante compromete el servidor web, no tiene acceso automático al DNS. Esto sigue el principio de separación de responsabilidades. En empresa real verás servidores dedicados solo a DNS, otros solo a web, otros solo a base de datos.

---

## ACLs — Control de Acceso entre VLANs

### ¿Qué son las ACLs?

Las ACLs (Access Control Lists) son el firewall del router Cisco. Definen qué tráfico puede pasar y qué tráfico se bloquea. Se aplican en una interfaz en dirección `in` (tráfico entrante) o `out` (tráfico saliente).

> 💼 **En trabajo real:** Las ACLs son la base del control de acceso en redes Cisco. En una entrevista de network o cybersecurity te preguntarán cómo aislarías una VLAN comprometida, o cómo evitarías que ventas acceda a servidores. La respuesta siempre involucra ACLs. Es pregunta estándar de CCNA y CompTIA Security+.

### Reglas de Seguridad Implementadas

| Origen | Destino | Acción | Razón |
|--------|---------|--------|-------|
| VLAN 20 Ventas | VLAN 10 Admin | BLOQUEAR | Ventas no debe ver datos de administración |
| VLAN 20 Ventas | VLAN 40 Servidores | BLOQUEAR | Acceso directo a servidores bloqueado |
| VLAN 20 Ventas | Server0 puerto 80 | PERMITIR | Ventas sí puede usar la intranet web |
| VLAN 30 IT | Todas las VLANs | PERMITIR | IT necesita acceso total para administrar |
| Cualquiera | VLAN 99 Management | BLOQUEAR | Solo IT puede administrar los dispositivos |
| VLAN 30 IT | VLAN 99 Management | PERMITIR | IT administra los switches y routers |

### ACL para Aislar Ventas

```
Router(config)# ip access-list extended VENTAS-RESTRICCIONES

! Bloquear ventas accediendo a administración
Router(config-ext-nacl)# deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255

! Bloquear ventas accediendo directo a servidores
Router(config-ext-nacl)# deny ip 192.168.20.0 0.0.0.255 192.168.40.0 0.0.0.255

! Permitir acceso web al servidor (puerto 80)
Router(config-ext-nacl)# permit tcp 192.168.20.0 0.0.0.255 host 192.168.40.20 eq 80

! Permitir todo lo demás (internet, DNS, etc)
Router(config-ext-nacl)# permit ip any any
Router(config-ext-nacl)# exit

! Aplicar la ACL en la subinterfaz de VENTAS
Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# ip access-group VENTAS-RESTRICCIONES in
Router(config-subif)# exit

! Verificar
Router# show ip access-lists
```

### ACL para Proteger VLAN de Management

```
! Solo IT (VLAN 30) puede acceder a VLAN 99
Router(config)# ip access-list extended PROTEGER-MANAGEMENT
Router(config-ext-nacl)# permit ip 192.168.30.0 0.0.0.255 192.168.99.0 0.0.0.255
Router(config-ext-nacl)# deny ip any 192.168.99.0 0.0.0.255
Router(config-ext-nacl)# permit ip any any
Router(config-ext-nacl)# exit

Router(config)# interface GigabitEthernet0/0.99
Router(config-subif)# ip access-group PROTEGER-MANAGEMENT in
```

> ⚠️ **Regla de oro de las ACLs:** Se evalúan de arriba hacia abajo y la primera coincidencia gana. Al final siempre hay un `deny any any` implícito. Por eso SIEMPRE debes poner `permit ip any any` al final si quieres dejar pasar el resto del tráfico. Si lo olvidas, bloqueas todo.

> **Wildcard mask:** En ACLs se usa wildcard mask (lo contrario de la máscara de subred). `/24` = `255.255.255.0` de máscara = `0.0.0.255` de wildcard. Los bits en 0 deben coincidir, los en 1 son "no importa".

---

## Verificación y Troubleshooting

### Checklist de Pruebas

| Prueba | Comando | Resultado Esperado |
|--------|---------|-------------------|
| DHCP funcionando | `ipconfig` en PC | IP en rango .100-.150 de su VLAN |
| Ping al gateway | `ping 192.168.X.1` | Reply from 192.168.X.1 |
| Inter-VLAN routing | Ping entre PCs de VLANs distintas | Reply (antes de ACLs) |
| DNS resuelve nombres | `nslookup intranet.techcorp.local` | Devuelve 192.168.40.20 |
| Intranet web funciona | `http://intranet.techcorp.local` | Página de TechCorp |
| ACL bloquea ventas | Ping desde VLAN20 a VLAN10 | Request timeout |
| IT accede a todo | Ping desde VLAN30 a todas las VLANs | Reply en todas |

### Comandos de Verificación — Router

```
Router# show ip route                ! Ver tabla de routing
Router# show ip dhcp binding         ! Ver IPs asignadas por DHCP
Router# show ip access-lists         ! Ver ACLs y matches por regla
Router# show ip interface brief      ! Ver estado de todas las interfaces
Router# show interfaces GigabitEthernet0/0.10  ! Ver subinterfaz específica
```

### Comandos de Verificación — Switch

```
Switch# show vlan brief              ! Ver todas las VLANs y sus puertos
Switch# show interfaces trunk        ! Ver puertos trunk activos
Switch# show interfaces FastEthernet0/1  ! Estado de una interfaz
Switch# show mac address-table       ! Ver qué dispositivo está en qué puerto
```

---

## Lecciones Aprendidas

- **VLANs** son segmentación lógica — permiten tener múltiples redes en un solo switch físico
- **Los servidores siempre tienen IP estática** — nunca DHCP en servidores de producción
- **Router-on-a-stick** permite inter-VLAN routing con un solo cable físico usando subinterfaces y 802.1Q
- **Las ACLs se evalúan de arriba hacia abajo** — el orden importa, siempre terminar con permit o deny explícito
- **Wildcard mask** en ACLs es lo inverso de la subnet mask: `0.0.0.255` = `/24`
- **DNS interno** usa dominio `.local` para separar nombres internos de internet público
- **VLAN 99 de management** existe para aislar la administración de dispositivos de red del resto de la red
- **Verificar siempre** con `show` commands después de cada configuración

---

*Network Lab — TechCorp S.A. | Cisco Packet Tracer*
