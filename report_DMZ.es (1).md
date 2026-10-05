
# Informe de configuración de DMZ con Cisco Packet Tracer


### 1. Objetivo del laboratorio

> Diseñar, implementar y validar una Zona Desmilitarizada (DMZ) segura en un router Cisco ISR (Router_FW) para publicar un servidor web interno hacia Internet mediante NAT estático, restringiendo de forma estricta el tráfico no autorizado (bloqueo de ICMP/ping desde el exterior y aislamiento de la red LAN respecto a la DMZ) mediante Listas de Control de Acceso (ACLs) extendidas.

**Ejemplo:**  
Configurar una DMZ segura usando un router Cisco ISR, aplicando NAT y ACLs para controlar el tráfico entre LAN, DMZ y red externa.

---

### 2. Topología implementada


- Cantidad de redes: 3 subredes IPv4 independientes
- Dispositivos usados: 1 Router Cisco ISR 2911, 3 Switches Cisco Catalyst 2960,2 Computadoras cliente, 1 Servidor web
- Breve descripción de la función de cada zona (LAN, DMZ, Externa). LAN Interna: Zona confiable de usuarios corporativos (PC_Internal). Requiere acceso al servidor de la DMZ para administración y consulta web, debiendo quedar completamente blindada de accesos iniciados desde la DMZ o el exterior.
DMZ (Zona Desmilitarizada): Zona semicolegiada donde reside el servidor web corporativo (Server-PT Web_DMZ). Expone servicios públicos pero tiene prohibido iniciar conexiones hacia la red interna.
Red Externa / Internet: Zona no confiable representada por PC_External. Solo tiene permitido el acceso web TCP (puerto 80) hacia la IP pública mapeada, con denegación total de tráfico de control (ICMP/ping).


### 3. Plan de direccionamiento IP

Completa la tabla con las IPs asignadas (puedes copiarla del enunciado si no cambió).

| Dispositivo             | IP              | Máscara           | Gateway           |
|-------------------------|------------------|-------------------|-------------------|
| PC_Internal             |  192.168.1.10    | 255.255.255.0     |192.168.1.1        |
| Server_DMZ                 192.168.2.10    | 255.255.255.0     |192.168.1.2        |
| PC_External             |  192.168.3.10    | 255.255.255.0     |192.168.1.3        |
| Router_FW Gi0/0 (LAN)   |  192.168.1.1     | 255.255.255.0     |    N/A            |
| Router_FW Gi0/1 (DMZ)   |  192.168.2.1     | 255.255.255.0     |    N/A            |
| Router_FW Gi0/1 (DMZ)   |  192.168.2.1     | 255.255.255.0     |    N/A            |
| Router_FW Gi0/2 (Ext)   |   192.168.3.1    | 255.255.255.0     |    N/A            |


### 4. Configuración aplicada (resumen)

> Resume los comandos o pasos más relevantes que ejecutaste. Usa texto + fragmentos de código cuando sea necesario.

- ip access-list extended EXTERNAL_IN
 permit tcp any host 192.168.3.1 eq 80
 permit tcp any host 192.168.2.10 eq 80
 deny ip any any
exit

interface GigabitEthernet0/2
 ip access-group EXTERNAL_IN in
exit
NAT Estático 1:1Mapeo directo de la dirección privada del servidor web hacia la dirección pública de la WAN:   
```ip nat inside source static 192.168.2.10 192.168.3.1
- ACLs: Permite tráfico TCP puerto 80 tanto hacia la IP pública como a la traducida, bloqueando el resto del tráfico proveniente de Internet (incluyendo ICMP).
```bash
ip access-list extended EXTERNAL_IN
 permit tcp any host 192.168.3.1 eq 80
 permit tcp any host 192.168.2.10 eq 80
 deny ip any any
exit

ACL DMZ_IN (Aplicada inbound en GigabitEthernet0/1):Permite el retorno de conexiones web establecidas (established) iniciadas legítimamente por la LAN interna, bloquea cualquier intento de nueva conexión originado desde la DMZ hacia la LAN, y permite el tráfico hacia otras redes (Internet).  

interface GigabitEthernet0/1
ip access-list extended DMZ_IN
 permit tcp host 192.168.2.10 eq 80 192.168.1.0 0.0.0.255 established
 permit tcp host 192.168.2.10 eq 443 192.168.1.0 0.0.0.255 established
 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
 permit ip any any
exit

interface GigabitEthernet0/1
 ip access-group DMZ_IN in
exit
```

### 5. Verificaciones realizadas

> Describe las pruebas y su resultado. Incluye capturas o salidas de comandos si se puede.
![alt text](image.png)
- `ping` desde PC_Internal al router: ✅
- Acceso web desde PC_External: ✅
- Bloqueo de acceso desde DMZ a LAN: ✅


### 6. Conclusiones y recomendaciones

> ¿Qué aprendiste con este ejercicio? ¿Qué mejorarías?
Orden de procesamiento NAT vs. ACLs: En Cisco IOS, el tráfico entrante en una interfaz con NAT exterior es evaluado por la ACL de entrada antes de que el motor de NAT traduzca las direcciones. La lista de control de acceso debe permitir el tráfico dirigido tanto a la dirección pública visible como a la IP interna final para evitar descartes en el simulador.
Manejo de sesiones y tráfico de retorno: El bloqueo estricto de una DMZ hacia la red interna no debe ser indiscriminado: el uso del parámetro established en la ACL permite que los paquetes de respuesta HTTP (SYN/ACK, ACK) regresen al cliente LAN sin permitir que un atacante que comprometa la DMZ pueda originar sesiones hacia la red corporativa.
Verificación metódica: Se recomienda validar primero el enrutamiento base e interfaces sin filtros, encender los servicios de aplicación (HTTP en el servidor DMZ) y luego aplicar las políticas de firewall capa por capa para aislar fallas de configuración.



### 7. Capturas de evidencia

> Adjunta aquí (o en un PDF anexo) las capturas solicitadas: pings, navegador, comandos `show`, etc.
![Ventana de Activity Results abierta en la pestaña Connectivity Tests mostrando las 6 pruebas en estado Correct.](image-1.png)
![Ventana de Activity Results en la pestaña Assessment Items mostrando el puntaje acumulado de Score: 9/9 y el mensaje de felicitaciones](image-2.png)
![Salida del comando show running-config en la CLI de Router_FW evidenciando las interfaces, las directivas NAT y las ACLs aplicadas.](image-3.png) ![](image-7.png) ![](image-8.png)