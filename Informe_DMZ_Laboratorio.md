### Informe de Laboratorio — Implementación de una DMZ Segura y Funcional

Estudiante: Diego Dario Silva Amorin
Fecha: 12/6/2026 
Herramienta: Cisco Packet Tracer  
Laboratorio: Building and Securing a Network with a DMZ  

---

### 1. Introducción

Una Zona Desmilitarizada (DMZ) es un segmento de red intermedio que separa los servicios 
públicos (como un servidor web) tanto de la red interna como de Internet. Su propósito es 
que, si un atacante compromete el servidor expuesto, no tenga acceso directo a la red LAN 
interna de la organización.

En este laboratorio se configuró una topología de tres zonas sobre un router Cisco ISR 2911 
actuando como firewall, aplicando NAT estático y ACLs para controlar el tráfico entre ellas.

---

###  2. Topología de red

El entorno consistió en tres segmentos de red conectados al router `Router_FW`:

| Zona | Interfaz del Router | Red |
|---|---|---|
| LAN Interna | GigabitEthernet0/0 | 192.168.1.0/24 |
| DMZ | GigabitEthernet0/1 | 192.168.2.0/24 |
| Red Externa | GigabitEthernet0/2 | 192.168.3.0/24 |

---

###  3. Plan de direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|---|---|---|---|---|
| Router_FW | Gi0/0 (LAN) | 192.168.1.1 | 255.255.255.0 | — |
| Router_FW | Gi0/1 (DMZ) | 192.168.2.1 | 255.255.255.0 | — |
| Router_FW | Gi0/2 (Externa) | 192.168.3.1 | 255.255.255.0 | — |
| PC_Internal | NIC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| Web_DMZ | NIC | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| PC_External | NIC | 192.168.3.10 | 255.255.255.0 | 192.168.3.1 |

---
###  4. Configuración del Router

4.1 Interfaces
Se asignaron direcciones IP a las tres interfaces del `Router_FW` y se activaron con 
`no shutdown`. Esto estableció la conectividad básica entre las tres zonas.

interface GigabitEthernet0/0

ip address 192.168.1.1 255.255.255.0

no shutdown
interface GigabitEthernet0/1

ip address 192.168.2.1 255.255.255.0

no shutdown
interface GigabitEthernet0/2

ip address 192.168.3.1 255.255.255.0

no shutdown

### 4.2 NAT Estático

El NAT estático permite que el servidor DMZ (IP privada `192.168.2.10`) sea accesible 
desde la red externa usando la IP pública `192.168.3.1`. Es una traducción uno a uno: 
cualquier tráfico que llegue a `192.168.3.1` desde afuera es redirigido internamente 
al servidor real.

interface GigabitEthernet0/1

ip nat inside
interface GigabitEthernet0/2

ip nat outside
ip nat inside source static 192.168.2.10 192.168.3.1

### 4.3 ACLs de Seguridad

Se implementaron dos ACLs con propósitos distintos:

**ACL 1 — Control de tráfico entrante desde Internet (aplicada en Gi0/2, inbound):**

Permite únicamente tráfico HTTP (puerto 80) hacia la IP pública del servidor. 
Cualquier otro tipo de tráfico desde Internet, incluido ICMP (ping), queda denegado 
por la regla implícita al final de toda ACL.

ip access-list extended ACL_EXTERNA

permit tcp any host 192.168.3.1 eq 80
interface GigabitEthernet0/2

ip access-group ACL_EXTERNA in

**ACL 2 — Bloqueo total de tráfico DMZ → LAN (aplicada en Gi0/1, inbound):**

Impide que cualquier dispositivo en la DMZ pueda iniciar comunicación hacia la red 
interna. Esto es crítico: si el servidor web fuera comprometido, el atacante no podría 
pivotar hacia la LAN.

ip access-list extended ACL_DMZ

deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255

permit ip any any
interface GigabitEthernet0/1

ip access-group ACL_DMZ in

---

## 5. Activación del servicio web

En el servidor `Web_DMZ`, dentro de la pestaña **Services → HTTP**, se verificó que 
los servicios HTTP y HTTPS estuvieran activos (`ON`), habilitando así la respuesta 
a solicitudes web.

---

## 6. Pruebas de validación

### 6.1 Conectividad básica (antes de ACLs)

| Prueba | Origen | Destino | Resultado |
|---|---|---|---|
| Ping gateway LAN | PC_Internal | 192.168.1.1 | ✅ Exitoso |
| Ping gateway DMZ | Web_DMZ | 192.168.2.1 | ✅ Exitoso |
| Ping gateway Externa | PC_External | 192.168.3.1 | ✅ Exitoso |

### 6.2 Acceso web

| Prueba | Origen | Destino | Resultado esperado |
|---|---|---|---|
| HTTP desde Internet | PC_External (browser) | 192.168.3.1 | ✅ Página web carga |
| HTTP desde LAN | PC_Internal (browser) | 192.168.2.10 | ✅ Página web carga |

### 6.3 Verificación de seguridad (con ACLs activas)

| Prueba | Origen | Destino | Resultado esperado |
|---|---|---|---|
| Ping desde Internet | PC_External | 192.168.3.1 | ❌ Request timed out |
| Ping DMZ → LAN | Web_DMZ | 192.168.1.10 | ❌ Request timed out |

*(Adjuntar capturas de pantalla en la carpeta `evidencias/`)*

---

## 7. Conclusiones

Este laboratorio demostró cómo una DMZ correctamente configurada permite exponer 
servicios públicos (un servidor web) de forma controlada, sin comprometer la red 
interna de la organización.

Los principales aprendizajes fueron:

- El **NAT estático** es la técnica que hace posible acceder a un servidor con IP 
  privada usando una dirección pública, ocultando la estructura interna de la red.
- Las **ACLs** actúan como un firewall de capa 3/4, filtrando tráfico según IP de 
  origen/destino y puerto. Su orden de aplicación (interfaz y dirección) es crítico 
  para que funcionen correctamente.
- La regla de **bloqueo DMZ → LAN** es la más importante desde el punto de vista 
  de seguridad: contiene el daño en caso de que el servidor sea comprometido.
- La **regla implícita `deny all`** al final de toda ACL en IOS es una capa de 
  seguridad adicional que hay que tener siempre presente al diseñar las reglas.