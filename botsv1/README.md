# Splunk BOTS v1: resumen del escenario y timeline del ataque

> **Estado:** laboratorio montado y dataset cargado. Investigación **en curso**: las secciones marcadas como *pendiente* se irán completando a medida que resuelva cada pregunta.
>
> **Aviso de spoilers:** los write-ups de este repositorio explican cómo se resuelven las preguntas del CTF. Si quieres jugarlo por tu cuenta, no los abras.

## 1. Contexto

BOTS v1 (*Boss of the SOC*) es un dataset de seguridad publicado por Splunk. Según la descripción oficial, recoge **dos ataques** atribuidos a un grupo hacktivista ficticio llamado **po1s0n1vy**, dirigidos contra **Wayne Corp / Wayne Enterprises**, una organización ficticia inspirada en el universo de Batman.

Los datos incluyen, además de la actividad maliciosa, **tráfico y eventos legítimos** (*white noise*), de modo que hay que distinguir lo anómalo entre grandes volúmenes de información, como en un SOC real.

| Escenario | Descripción general |
|---|---|
| **1. Defacement web** | Se informa de que el sitio web `imreallynotbatman.com`, alojado en el espacio de direcciones de la empresa, ha sido comprometido. Hay que determinar qué ocurrió y reconstruir la intrusión. |
| **2. Ransomware** | Un equipo interno resulta infectado por el ransomware **Cerber**. Hay que determinar cómo se produjo la infección y su alcance. |

Mis resultados, hallazgos y razonamientos se irán añadiendo en los apartados siguientes.

## 2. Entorno y datos

- Índice de Splunk: `index=botsv1` (se consulta con `earliest=0`, porque los eventos son de 2016).
- Dataset completo (Splunk Indexed): 6,1 GB comprimido, ~8,63 GB descomprimido.
- Montaje del laboratorio y problemas encontrados: ver [`../lab-setup/`](../lab-setup/).
- Los datos pertenecen a Splunk y se publican con licencia CC0 según el repositorio oficial. **No se incluyen en este repositorio.**

### Fuentes de datos (sourcetypes)

| Sourcetype | Qué contiene |
|---|---|
| `WinEventLog:Application`, `:Security`, `:System` | Registros de eventos de Windows |
| `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` | Actividad detallada de procesos, red y archivos (Sysmon) |
| `winregistry` | Cambios en el registro de Windows |
| `fgt_event`, `fgt_traffic`, `fgt_utm` | Logs del firewall Fortinet FortiGate: eventos, tráfico y seguridad (UTM) |
| `suricata` | Alertas y eventos del IDS Suricata |
| `iis` | Logs del servidor web IIS |
| `nessus:scan` | Resultados de escaneos de vulnerabilidades |
| `stream:dns`, `stream:http`, `stream:dhcp`, `stream:smb`, `stream:tcp`, `stream:ip`, `stream:icmp`, `stream:ldap`, `stream:mapi`, `stream:sip`, `stream:snmp` | Tráfico de red capturado con Splunk Stream, por protocolo |

## 3. Metodología

- Cada escenario se estructura seguiendo relativamente la **Cyber Kill Chain**.
- Cada hallazgo se documenta con su pregunta, razonamiento, consulta SPL, evidencia y conclusión.
- Las consultas reutilizables se guardan en [`../spl/`]/spl/, las detecciones propias en [`/detections`]/detections/ y los paneles visuales en [`/dashboards/`]/dashboards/.

## 4. Progreso

| Escenario | Estado | Write-up |
|---|---|---|
| 1. Defacement web | Completado | Hecho |
| 2. Ransomware | Pendiente | — |

## 5. Estructura de esta carpeta (prevista)

```
botsv1/
├── README.md       # este documento
├── writeups/       # un archivo por pregunta o por fase
├── spl/            # consultas reutilizables, comentadas
├── detections/     # reglas y alertas propias
└── dashboards/     # exports de dashboards y capturas
```
