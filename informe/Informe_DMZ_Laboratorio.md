1. Objetivo del laboratorioDiseñar, implementar y asegurar una Zona Desmilitarizada (DMZ) en Cisco Packet Tracer utilizando un router Cisco ISR 2911 como firewall. El propósito fundamental es publicar un servicio Web (HTTP) de forma segura hacia Internet mediante NAT estático, restringiendo simultáneamente el acceso no autorizado hacia la red LAN interna mediante Listas de Control de Acceso (ACLs).2. Topología implementadaCantidad de redes: 3 subredes independientes (LAN, DMZ y Externa/Internet).Dispositivos usados: 1x Router Cisco ISR 2911 (Router_FW), 3x Switches Cisco 2960 (SW_Internal, SW_DMZ, SW_External), 2x PCs (PC_Internal, PC_External) y 1x Servidor (Server-PT Web_DMZ).Descripción de zonas:Red LAN (Interna): Segmento seguro que alberga a los usuarios de la organización (PC_Internal).Red DMZ (Zona Desmilitarizada): Segmento aislado donde reside el servidor web público (Server-PT Web_DMZ).Red Externa (Internet): Representa clientes externos o no confiables intentando acceder a los servicios públicos (PC_External).3. Plan de direccionamiento IP  DispositivoIPMáscaraGatewayPC_Internal192.168.1.10255.255.255.0192.168.1.1Server_DMZ192.168.2.10255.255.255.0192.168.2.1PC_External192.168.3.10255.255.255.0192.168.3.1Router_FW Gi0/0 (LAN)192.168.1.1255.255.255.0N/ARouter_FW Gi0/1 (DMZ)192.168.2.1255.255.255.0N/ARouter_FW Gi0/2 (Ext)192.168.3.1255.255.255.0N/A4. Configuración aplicada (resumen)  Interfaces y Marcas de NAT:Plaintextinterface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown
exit

interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 ip nat inside
 no shutdown
exit

interface GigabitEthernet0/2
 ip address 192.168.3.1 255.255.255.0
 ip nat outside
 no shutdown
exit
Mapeo de NAT Estático:Plaintextip nat inside source static 192.168.2.10 192.168.3.1
Listas de Control de Acceso (ACLs Extendidas):Plaintext! Permitir trafico HTTP entrante a la IP publica publicada
ip access-list extended EXTERNAL_IN
 permit tcp any host 192.168.3.1 eq 80
exit

! Proteger la red interna LAN de cualquier iniciacion de trafico desde la DMZ
ip access-list extended DMZ_PROTECT
 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
 permit ip any any
exit

! Aplicación en interfaces
interface GigabitEthernet0/2
 ip access-group EXTERNAL_IN in
exit

interface GigabitEthernet0/1
 ip access-group DMZ_PROTECT in
exit
5. Verificaciones realizadasping 192.168.1.1 desde PC_Internal: ✅ Exitoso (Respuesta correcta del gateway local).ping 192.168.2.1 desde Web_DMZ: ✅ Exitoso (Respuesta correcta del gateway local).Acceso Web a [http://192.168.3.1](http://192.168.3.1) desde PC_External: ✅ Exitoso (La traducción NAT redirige el puerto 80 hacia el servidor de la DMZ).Acceso Web a [http://192.168.2.10](http://192.168.2.10) desde PC_Internal: ✅ Exitoso (Navegación interna funcional).ping 192.168.1.10 desde Web_DMZ: ✅ Bloqueado (Request timed out, verificado por la regla DMZ_PROTECT).ping 192.168.3.1 desde PC_External: ✅ Bloqueado (Request timed out, ICMP denegado por la ACL EXTERNAL_IN).6. Conclusiones y recomendacionesSe comprendió la importancia defensiva de aislar servidores expuestos a la red pública dentro de una zona DMZ para prevenir movimientos laterales de un atacante.La combinación de NAT Estático y ACLs extendidas permite exponer únicamente los puertos indispensables (TCP 80), manteniendo una postura de menor privilegio.Se recomienda utilizar la herramienta Fast Forward Time en Packet Tracer para forzar la convergencia de protocolos de red antes de ejecutar las evaluaciones de conectividad.7. Capturas de evidencia(Adjunta en la carpeta evidencias/ de tu repositorio las imágenes exportadas de las pruebas de navegador y la ventana Check Results)
