# Detections – Escenario 1: Defacement (BOTS v1)

Este documento recoge las detecciones y búsquedas de threat hunting desarrolladas a partir de la investigación del escenario 1 de BOTS v1. El objetivo es transformar los hallazgos del writeup en lógica de detección reutilizable en Splunk.

## Resumen de detecciones

| ID | Nombre | Fuente principal | Tipo | MITRE ATT&CK |
|---|---|---|---|---|
| DET-001 | Web Vulnerability Scanning | Splunk Stream HTTP | Detección de comportamiento | T1595 – Active Scanning |
| DET-002 | Joomla Brute Force | Splunk Stream HTTP | Detección de comportamiento | T1110 – Brute Force |
| DET-003 | Suspicious Joomla Admin Activity | Splunk Stream HTTP | Detección de actividad administrativa sospechosa | No asignada: el envío de un payload no demuestra su ejecución |
| DET-004 | Suspicious Executable Transfer to Web Server | Suricata | Detección de transferencia | T1105 – Ingress Tool Transfer |
| DET-005 | Suspicious Process Execution from Web Directory | Sysmon | Detección de ejecución sospechosa | No asignada: la ruta por sí sola no permite determinar una técnica ATT&CK concreta |
| HUNT-001 | Suspicious Defacement Resource Retrieval | Splunk Stream HTTP / Suricata / FortiGate | Búsqueda de threat hunting | No asignada: la actividad es compatible con un intento de defacement, no con un defacement confirmado |

---

## DET-001 – Web Vulnerability Scanning

### Descripción
Identifica posibles actividades automatizadas de reconocimiento y escaneo contra una aplicación web mediante un volumen elevado de solicitudes HTTP y el acceso a numerosas rutas diferentes desde una misma dirección IP.

### Objetivo
Detectar comportamientos compatibles con herramientas de escaneo de vulnerabilidades web. En el escenario investigado se observó actividad desde `40.80.148.42` y cabeceras HTTP que identificaban Acunetix Web Vulnerability Scanner 10.0.

### Fuente de datos
- Índice: `botsv1` (adaptar al entorno).
- Sourcetype: `stream:http`.
- Campos: `src_ip`, `uri_path`, `_time`, `http_method`, `http_user_agent` o `src_headers` (según los campos disponibles).

### Lógica de detección
Agrupa las solicitudes HTTP por IP de origen en intervalos de cinco minutos y busca un volumen elevado de peticiones junto con diversidad de rutas solicitadas.

### Evidencia del escenario
La investigación identificó actividad HTTP de reconocimiento contra `imreallynotbatman.com`, incluyendo solicitudes a múltiples rutas y cabeceras que identificaban Acunetix Web Vulnerability Scanner 10.0.

### MITRE ATT&CK
- **T1595 – Active Scanning.** La técnica es pertinente cuando la actividad observada corresponde a sondeo activo de sistemas o servicios.

---

## DET-002 – Joomla Brute Force

### Descripción
Detecta un volumen elevado de solicitudes POST dirigidas al formulario de autenticación del panel administrativo de Joomla desde una misma IP.

### Objetivo
Identificar posibles intentos automatizados de adivinación de credenciales contra `/joomla/administrator/index.php`.

### Fuente de datos
- Índice: `botsv1` (adaptar al entorno).
- Sourcetype: `stream:http`.
- Campos: `src_ip`, `uri_path`, `http_method`, `form_data`, `status`, `_time`.

### Lógica de detección
Cuenta las solicitudes POST al endpoint de administración por IP de origen durante una ventana de cinco minutos. La detección se basa en el volumen de intentos, no en asumir que un código HTTP concreto representa necesariamente un fallo de autenticación.

### Evidencia del escenario
Se identificaron 412 peticiones POST desde `23.22.63.114` al panel de administración de Joomla. También se observó actividad desde `40.80.148.42`, que posteriormente realizó operaciones administrativas asociadas a una misma sesión.

### MITRE ATT&CK
- **T1110 – Brute Force.** Aplicable a intentos repetidos de adivinación de credenciales.

---

## DET-003 – Suspicious Joomla Admin Activity

### Descripción
Identifica solicitudes POST al panel administrativo de Joomla que contienen operaciones sensibles relacionadas con la instalación de paquetes, en particular el parámetro `install_package`.

### Objetivo
Alertar sobre actividad administrativa que puede utilizarse para introducir extensiones, archivos o código no autorizado en una aplicación Joomla.

### Fuente de datos
- Índice: `botsv1` (adaptar al entorno).
- Sourcetype: `stream:http`.
- Campos: `src_ip`, `uri_path`, `http_method`, `form_data`, `status`, `cookie`, `_time`.

### Lógica de detección
Busca peticiones POST al endpoint administrativo cuyo cuerpo incluya `install_package`. Se trata de una detección específica de una operación sensible; no confirma que el paquete se haya instalado ni que el código enviado se haya ejecutado.

### Evidencia del escenario
Desde `40.80.148.42` se observaron peticiones POST al panel de Joomla que incluían `install_package` y operaciones administrativas del componente `com_explorer`. En el contenido de una de las peticiones se identificó código PHP ofuscado.

### MITRE ATT&CK
No se asigna una técnica específica en esta versión. Aunque la actividad puede ser compatible con la introducción de una web shell u otro mecanismo de persistencia, los registros analizados demuestran el envío de una petición administrativa y un payload, no la instalación o ejecución efectiva de una web shell.

---

## DET-004 – Suspicious Executable Transfer to Web Server

### Descripción
Identifica archivos ejecutables detectados por Suricata durante transferencias de red, con especial atención a los que se dirigen a servidores web.

### Objetivo
Detectar la transferencia de ejecutables hacia servidores que alojan aplicaciones web, para facilitar la investigación de posibles herramientas o cargas maliciosas.

### Fuente de datos
- Índice: `botsv1` (adaptar al entorno).
- Sourcetype: `suricata`.
- Campos: `event_type`, `fileinfo.filename`, `fileinfo.size`, `fileinfo.state`, `fileinfo.stored`, `src_ip`, `dest_ip`, `dest_port`, `_time`.

### Lógica de detección
Busca eventos `fileinfo` en los que el nombre del archivo termine en `.exe`. Los metadatos de Suricata permiten identificar el nombre, tamaño, origen, destino y estado de almacenamiento del archivo.

### Evidencia del escenario
Suricata identificó la transferencia de `3791.exe`, de 2.705 bytes, desde `40.80.148.42` hacia `192.168.250.70` a través del puerto 80. El evento de `fileinfo` indicaba `stored=false`, por lo que Suricata no almacenó el contenido del archivo en ese registro.

### MITRE ATT&CK
- **T1105 – Ingress Tool Transfer.** Aplicable cuando se confirma la transferencia de una herramienta o archivo al sistema comprometido. En el escenario se observó la transferencia de `3791.exe` al servidor web.

---

## DET-005 – Suspicious Process Execution from Web Directory

### Descripción
Identifica la creación de procesos ejecutables desde directorios utilizados para alojar aplicaciones web, como `C:\inetpub\wwwroot\`.

### Objetivo
Detectar la ejecución de archivos desde directorios web, comportamiento que puede indicar que un atacante ha conseguido colocar y ejecutar un archivo en el servidor.

### Fuente de datos
- Índice: `botsv1` (adaptar al entorno).
- Sourcetype: `XmlWinEventLog` (Sysmon).
- Evento: `EventCode=1` – Process Create.
- Campos: `Image`, `CommandLine`, `ParentImage`, `ProcessId`, `ProcessGuid`, `User`, `host`, `_time`.

### Lógica de detección
Busca eventos Sysmon de creación de procesos cuya ruta `Image` se encuentre bajo el directorio web de IIS.

### Evidencia del escenario
Sysmon EventCode 1 registró la ejecución de `C:\inetpub\wwwroot\joomla\3791.exe` en `we1149srv`. El proceso tenía como proceso padre `cmd.exe` y se ejecutó bajo la cuenta `NT AUTHORITY\IUSR`.

La correlación mediante `ProcessGuid` permitió identificar eventos adicionales: una conexión de red (EventCode 3), 61 eventos de carga de imágenes (EventCode 7) y la terminación del proceso (EventCode 5). La conexión registrada se dirigía desde `192.168.250.70:63394` hacia `23.22.63.114:3791`.

### MITRE ATT&CK
No se asigna una técnica específica en esta versión. La ejecución de un proceso desde un directorio web es sospechosa, pero no basta por sí sola para determinar el mecanismo de ejecución o la técnica utilizada.

---

## HUNT-001 – Suspicious Defacement Resource Retrieval

### Tipo
Búsqueda de threat hunting / investigación retrospectiva. No se presenta como una detección de defacement completado.

### Descripción
Busca solicitudes HTTP realizadas desde un servidor web hacia recursos externos que puedan estar relacionados con una modificación no autorizada del contenido del sitio.

### Objetivo
Identificar solicitudes salientes desde servidores web hacia imágenes u otros recursos externos sospechosos y comprobar si existen evidencias adicionales en Suricata, FortiGate e IIS.

### Fuente de datos
- `stream:http`: solicitudes HTTP y metadatos de la comunicación.
- `suricata`: eventos HTTP y `fileinfo`.
- FortiGate: registros de tráfico y clasificación de destinos.
- `iis`: solicitudes recibidas por el servidor web.

### Evidencia del escenario
Se identificaron dos solicitudes GET desde `192.168.250.70` hacia `23.22.63.114` para obtener `poisonivy-is-coming-for-you-batman.jpeg`, asociado al dominio `prankglassinebracket.jumpingcrab.com`.

Los eventos de Stream HTTP mostraban `bytes_out=0`, `response_time=0` y `canceled=1`, por lo que esos eventos, considerados aisladamente, no demostraban la recepción completa del recurso. En otros registros, Suricata identificó una respuesta HTTP 200 con tipo `image/jpeg` y longitud indicada de 553.879 bytes. El evento `fileinfo` reflejaba `state=TRUNCATED` y `stored=false`. FortiGate registró la conexión como permitida (`action=passthrough`) y clasificó el destino en la categoría `Malicious Websites`.

La revisión de IIS no permitió corroborar que el contenido modificado llegara a servirse a los visitantes.

### MITRE ATT&CK
No se asigna una técnica específica. La actividad investigada es compatible con un intento de defacement, pero no se ha confirmado que la página fuera modificada y servida públicamente.

### Conclusión de la investigación
Se confirma que el servidor realizó solicitudes relacionadas con la imagen y que existen registros adicionales de Suricata y FortiGate sobre la comunicación. Sin embargo, no se ha podido confirmar que la imagen llegara a mostrarse públicamente. Por ello, esta búsqueda se mantiene como una consulta de threat hunting y no como una alerta de defacement confirmado.

---

### Relación con el writeup
Estas detecciones se derivan de los hallazgos documentados en el writeup del escenario 1:
- Reconocimiento web y escaneo de rutas.
- Intentos de autenticación contra Joomla.
- Actividad administrativa sospechosa.
- Transferencia de `3791.exe`.
- Ejecución del archivo y actividad de red asociada.
- Solicitudes relacionadas con la imagen del posible defacement.

La atribución contextual del escenario a `po1s0n1vy` no se utiliza como condición de ninguna detección: las reglas se basan en comportamientos e indicadores observables, no en una atribución de actor no confirmada.
