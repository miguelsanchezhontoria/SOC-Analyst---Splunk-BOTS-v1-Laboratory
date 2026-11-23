## Contexto
Cada dashboard cuenta con su código en formato JSON como una imagen para poder verlo de manera más visual. Cada uno de estos archivos están en esta carpeta (en total son 6, dos por dashboard). Los paneles más comunes son tablas, gráficos de barras y gráficos circulares, aunque hay otros formatos también. Al incluir esta carpeta se pretende tener un apoyo más "sencillo" para poder comprender el escenario desde otro punto de vista.

## 1. Web Attack Overview
Panel 1. HTTP Requests Over Time -> Muestra la evolución temporal de las peticiones HTTP.\
Panel 2. Top Source IPs -> Identifica las direcciones IP que más peticiones HTTP realizaron al dominio investigado.\
Panel 3. Most Requested URI Paths -> Muestra las rutas del sitio web que recibieron más solicitudes.\
Panel 4. HTTP Status Codes -> Representa la distribución de códigos de respuesta HTTP.\
Panel 5. Suricata Alerts by Signature -> Muestra las firmas de Suricata más frecuentes relacionadas con el dominio investigado.\
Panel 6. HTTP Activity by Source IP -> Muestra un resumen de actividad por IP de origen.\

## 2. Joomla Authentication & Admin Activity
Panel 1. Joomla Login Attempts Over Time -> Muestra la evolución temporal de las peticiones POST al panel de administración.\
Panel 2. Login Attempts by Source IP -> Identifica las IP que más peticiones POST realizaron al formulario de Joomla.\
Panel 3. Credentials Observed -> Extrae las combinaciones de usuario y contraseña observadas en las peticiones POST.\
Panel 4. HTTP Response Codes -> Muestra los códigos HTTP asociados a los intentos de autenticación.\
Panel 5. Suspicious Joomla Admin Operations -> Identifica operaciones administrativas relacionadas con la instalación de paquetes.\
Panel 6. Administrative Activity by Source IP -> Resume la actividad administrativa de cada IP.\

## 3. Web Server Compromise
Panel 1. Executable Files Detected -> Identifica archivos ejecutables detectados por Suricata.\
Panel 2. Process Execution on Web Server -> Muestra procesos ejecutados desde el directorio web de IIS.\
Panel 3. Process Executions Over Time -> Muestra la evolución temporal de los procesos creados en el servidor web.\
Panel 4. Outbound Network Connections -> Identifica conexiones de red registradas por Sysmon EventCode 3.\
Panel 5. Loaded Modules by Process -> Muestra las bibliotecas y módulos cargados por los procesos mediante Sysmon EventCode 7.\
Panel 6. Sysmon Events by EventCode -> Resume los eventos Sysmon registrados en el servidor.\