# Laboratorio: Implementación de una DMZ Segura y Funcional

Este repositorio contiene la topología, configuraciones y documentación técnica del laboratorio práctico de redes y seguridad perimetral desarrollado en Cisco Packet Tracer.

---

## 🎯 Objetivo del Laboratorio

Diseñar, configurar y validar una Zona Desmilitarizada (DMZ) en un router Cisco ISR (Router_FW) para publicar de forma segura un servidor web interno hacia Internet mediante NAT estático, aplicando Listas de Control de Acceso (ACLs) extendidas para mitigar vectores de ataque y aislar la red corporativa interna.

---

## 📂 Contenido del Repositorio

| Archivo / Carpeta | Descripción |
| :--- | :--- |
| `DMZ_Laboratorio.pka` | Archivo de topología y simulación resuelto al 100% en Cisco Packet Tracer. |
| `INFORME.md` / `Informe_DMZ.pdf` | Informe técnico detallado con topología, plan de direccionamiento, configuraciones aplicadas y evidencias de conectividad. |
| `capturas/` | Carpeta de imágenes con evidencias de validación (Connectivity Tests 6/6, Assessment Items 9/9 y CLI del router). |

---

## 🏗️ Resumen de la Topología y Segmentación

* **LAN Interna (`192.168.1.0/24`):** Red confiable donde opera `PC_Internal` (`192.168.1.10`). Accede a la DMZ pero queda completamente aislada ante conexiones entrantes.
* **DMZ (`192.168.2.0/24`):** Segmento intermedio donde reside el servidor web `Server-PT Web_DMZ` (`192.168.2.10`), accesible públicamente vía HTTP.
* **Red Externa (`192.168.3.0/24`):** Red pública simulada donde opera `PC_External` (`192.168.3.10`).

---

## 🔐 Políticas de Seguridad Implementadas

1. **NAT Estático 1:1:** Mapeo de la IP privada del servidor web (`192.168.2.10`) a la IP de la interfaz WAN (`192.168.3.1`) para exponer el servicio HTTP (puerto 80) a Internet.
2. **ACL Externa (`EXTERNAL_IN`):** Permite exclusivamente tráfico TCP en el puerto 80 hacia el servidor web y deniega el resto del tráfico proveniente de la red externa, bloqueando ataques de reconocimiento ICMP (ping).
3. **ACL DMZ (`DMZ_IN`):** Aplica filtrado *stateful* básico mediante la directiva `established` para permitir únicamente los retornos de paquetes HTTP hacia la LAN iniciados por clientes legítimos, bloqueando cualquier intento de inicio de conexión originado desde la DMZ hacia la red interna.

---

## ✅ Resultados de Evaluación

* **Score:** 9/9 (100%)
* **Connectivity Tests:** 6/6 Pruebas superadas con éxito (ICMP de gestión permitido, navegación web funcional desde LAN y WAN, y bloqueos de seguridad activos).
