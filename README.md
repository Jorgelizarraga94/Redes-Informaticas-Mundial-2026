# 🏟️ Infraestructura de Red Centralizada - Mundial 2026

Simulación en Cisco Packet Tracer de una infraestructura de red escalable y de gran densidad para un torneo distribuido en 3 regiones geográficas (**Canadá, EEUU y México**), administrando un total de 6 estadios.

La arquitectura implementa segmentación lógica por VLANs, optimización del direccionamiento IPv4 mediante VLSM y automatización de servicios esenciales (DHCP y DNS centralizado).

---

## 🚀 Requisitos para la Ejecución

Para visualizar y testear la topología interactiva, es necesario utilizar el simulador oficial de Cisco.

* **Software:** Cisco Packet Tracer
* **Versión Mínima:** **9.0** (o superior)
* **Archivo:** `.pkt`

> ⚠️ **Nota de Compatibilidad:** Abrir el proyecto en versiones anteriores a la 9.0 (como 8.2) romperá las tablas de enrutamiento y la sintaxis de los scripts DHCP debido a incompatibilidades del IOS.

---

## 📐 Cuestiones Clave del Diseño

### 1. Justificación del Direccionamiento (VLSM)
Para cubrir el requerimiento masivo de hosts ($150.000$ por estadio para el público) sin agotar el espacio IPv4, se migró de un esquema inicial fijo `/13` a un bloque mayor **`/12` solicitando 1 bit prestado**. Esto permitió asignar una **máscara `/13` fija por país** con 8 saltos en el tercer octeto, optimizando el espacio útil.

### 2. Segmentación Funcional (VLANs)
El tráfico interno de cada sede se aisló a través de subinterfaces en los routers core (`Router-on-a-Stick`) bajo el estándar **IEEE 802.1Q**:
* **VLAN 10 (Público):** Bloques `/14` para alta densidad de usuarios.
* **VLAN 20 (Deportistas):** Bloques `/23` privados para delegaciones.
* **VLAN 30 (Logística):** Bloques `/24` para la gestión operativa.
* **VLAN 40 (Servidores):** Bloques `/27` para la infraestructura crítica.

### 3. Automatización de Servicios (DHCP y DNS)
* **DHCP Nativo:** Configurado en los Routers Core de cada región, entregando parámetros dinámicos según la subinterfaz lógica de origen.
* **Exclusiones:** Se reservaron de forma estática las primeras IPs útiles (de la `.1` a la `.10` / `.5`) para proteger Gateways e infraestructura crítica.
* **DNS Centralizado:** Todos los pools inyectan de forma automatizada la IP **`10.4.3.4`** (alojada en la VLAN de Servidores de Vancouver), unificando la resolución de nombres del torneo.

---

## 🌐 Cuadro Resumen de Direccionamiento

* **Canadá (Bloque `10.0.0.0 /13`)**
  * **Vancouver:** Redes desde `10.0.0.0 /14` hasta Servidores en `10.4.3.0 /27`.
  * **Toronto:** Redes desde `10.8.0.0 /14` hasta Servidores en `10.12.3.0 /27`.
* **USA (Bloque `10.16.0.0 /13`)**
  * **San Francisco:** Redes desde `10.16.0.0 /14` hasta Servidores en `10.20.3.0 /27`.
  * **Nueva York:** Redes desde `10.24.0.0 /14` hasta Servidores en `10.28.3.0 /27`.
* **México (Bloque `10.32.0.0 /13`)**
  * **Monterrey:** Redes desde `10.32.0.0 /14` hasta Servidores en `10.36.3.0 /27`.
  * **Guadalajara:** Redes desde `10.40.0.0 /14` hasta Servidores en `10.44.3.0 /27`.
* **Enlaces WAN Core-to-Core:** Conexiones seriales punto a punto configuradas con máscaras **`/30`** (ej. `10.100.0.1/30`) para evitar el desperdicio de direccionamiento.

---

## 🧪 Comandos de Verificación en la CLI

Para auditar la conectividad desde el modo privilegiado (`#`) de los routers:
* **Estado de Interfaces:** `show ip interface brief` (Valida que las subinterfaces y seriales estén `up/up`).
* **Tablas de Rutas:** `show ip route` (Verifica la convergencia de caminos hacia los demás países).
* **Prueba de Conectividad:** `ping [IP]` (Representado con éxito mediante la secuencia **`!!!!!`**).
* **Trazabilidad Extremo a Extremo:** `traceroute 10.4.3.4` (Sigue el salto de paquetes inter-vlan hacia el servidor DNS central).

---

<img width="1401" height="724" alt="redes" src="https://github.com/user-attachments/assets/f20e806b-e71e-4801-9d1d-f577f1576613" />


## 📝 Conclusión

El proyecto permitió consolidar la teoría de redes en un escenario corporativo/deportivo de gran escala. La implementación de **VLANs** y la optimización con **VLSM** demostraron la importancia de una distribución eficiente del direccionamiento IPv4, mientras que el despliegue de **DHCP** evidenció cómo la automatización de infraestructura mitiga errores humanos y agiliza el aprovisionamiento de miles de hosts distribuidos geográficamente.
