DMZ Lab — Configuración y Seguridad de Red con Zona Desmilitarizada

Laboratorio práctico de ciberseguridad defensiva realizado en Cisco Packet Tracer.

Objetivo

Configurar una red segmentada en tres zonas (LAN, DMZ, Externa) aplicando NAT estático
y ACLs para controlar el tráfico entre ellas, simulando un entorno real con firewall.

Contenido del repositorio

| `DMZ_PROJECT.pka` | Archivo Packet Tracer con la configuración completa |
| `informe/Informe_DMZ_Laboratorio.md` | Informe técnico del laboratorio |
 Capturas de pantalla de las pruebas de validación |

Topología

- **Router_FW** (ISR 2911): actúa como firewall entre las tres zonas  
- **LAN** (192.168.1.0/24): red interna con PC_Internal  
- **DMZ** (192.168.2.0/24): servidor web Web_DMZ  
- **Red Externa** (200.0.0.0/24): simula internet con PC_External
- 
Pruebas realizadas

- Ping exitoso desde LAN al servidor DMZ  
- Acceso HTTP desde red externa al servidor DMZ (vía NAT estático)  
- Bloqueo confirmado de tráfico desde DMZ hacia la LAN
