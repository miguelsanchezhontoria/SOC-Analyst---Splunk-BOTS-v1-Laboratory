# Cheatsheet de Sysmon

Chuleta de los eventos de Sysmon (herramienta de Microsoft que registra actividad detallada de procesos, red y archivos) utilizados en el laboratorio. Los eventos se almacenan en el sourcetype `XmlWinEventLog`.

## Eventos documentados

| Event ID | Nombre | Qué registra |
|---:|---|---|---|
| 1 | Process Creation | Creación de un proceso | 
| 3 | Network Connection | Conexión de red iniciada por un proceso | 
| 5 | Process Terminated | Finalización de un proceso | 
| 7 | Image Loaded | Carga de un módulo o biblioteca (DLL) | 

Los campos y ejemplos de cada evento se detallan abajo. 

## Event ID 1: Process Creation

Registra la creación de un proceso. Permite saber qué ejecutable se inició, con qué argumentos y qué proceso lo lanzó.

| Campo | Qué contiene |
|---|---|
| `Image` | Ruta del ejecutable iniciado |
| `CommandLine` | Línea de comandos utilizada |
| `ParentImage` | Proceso padre que lo inició |
| `ProcessId` | Identificador del proceso (PID) |
| `ProcessGuid` | Identificador único del proceso, útil para correlacionar sus eventos |
| `User` | Usuario bajo el que se ejecutó |
| `Hashes` | Hashes del ejecutable: SHA1, MD5, SHA256 e IMPHASH. Son la base para buscar el archivo en fuentes externas |

**Aplicación en el escenario 1:** permitió identificar la ejecución de `3791.exe` en `we1149srv`, lanzado por `cmd.exe` bajo la cuenta `NT AUTHORITY\IUSR`. Esta cuenta es la que IIS utiliza para el acceso anónimo, lo que sitúa el origen de la ejecución en el servidor web. Además, el campo `Hashes` proporcionó los indicadores de compromiso del archivo.

## Event ID 3: Network Connection

Registra las conexiones de red iniciadas por un proceso.

| Campo | Qué contiene |
|---|---|
| `Protocol` | Protocolo de transporte (por ejemplo, TCP) |
| `Initiated` | Si la conexión la inició el propio proceso (`true`) |
| `SourceIp` / `SourcePort` | Origen de la conexión |
| `DestinationIp` / `DestinationPort` | Destino de la conexión |
| `DestinationHostname` | Nombre resuelto del destino, si está disponible |

**Aplicación en el escenario 1:** atribuyó a `3791.exe` una conexión TCP desde `192.168.250.70:63394` hacia `23.22.63.114:3791`, con nombre de destino `ec2-23-22-63-114.compute-1.amazonaws.com`. Se correlacionó mediante el `ProcessGuid` y relaciona el ejecutable con la infraestructura del atacante.

## Event ID 5: Process Terminated

Registra la finalización de un proceso.

**Aplicación en el escenario 1:** apareció entre los eventos asociados al `ProcessGuid` investigado. No se realizó un análisis individual de este evento.

## Event ID 7: Image Loaded

Registra la carga de módulos o bibliotecas (DLL) dentro de un proceso.

| Campo | Qué contiene |
|---|---|
| `Image` | Proceso que carga el módulo |
| `ImageLoaded` | Ruta del módulo o biblioteca cargada |
| `ProcessId` / `ProcessGuid` | Identificadores del proceso |
| `Hashes` | Hashes del módulo |
| `Signed` / `Signature` | Si el archivo está firmado y datos de la firma |
| `User` | Usuario asociado al proceso |

**Aplicación en el escenario 1:** permitió observar las bibliotecas cargadas por `3791.exe`, como `wsock32.dll` y `mswsock.dll` (relacionadas con comunicaciones de red) y otras bibliotecas del sistema.

## Correlación de eventos mediante `ProcessGuid`

El campo `ProcessGuid` relaciona los distintos eventos de Sysmon de un mismo proceso, aunque tengan `EventCode` diferentes. Es la forma de seguir un proceso desde su creación hasta el final de su actividad.

**Flujo de trabajo utilizado:**

1. Buscar el nombre del archivo en todo el índice para localizar eventos relacionados:

   ```spl
   index=botsv1 earliest=0 "<archivo>"
   | table _time host sourcetype EventCode Image TargetFilename CommandLine ParentImage
   | sort 0 _time
   ```

2. Obtener el `ProcessGuid` del evento de creación (ID 1).
3. Resumir qué tipos de evento tiene ese proceso:

   ```spl
   index=botsv1 earliest=0 sourcetype=XmlWinEventLog ProcessGuid="{<GUID>}"
   | stats count by EventCode
   | sort EventCode
   ```

4. Examinar cada tipo de evento relevante: ID 3 para la red, ID 7 para los módulos cargados, etc.

**Resultado en el escenario 1** (`ProcessGuid` `{E500B0EA-A302-57AB-0000-00108D65C301}`):

| Event ID | Tipo de evento | Cantidad |
|---|---|---:|
| 1 | Process Creation | 1 |
| 3 | Network Connection | 1 |
| 5 | Process Terminated | 1 |
| 7 | Image Loaded | 61 |

**Precaución:** para afirmar el orden de los eventos (por ejemplo, creación, conexión de red y finalización) hay que ordenarlos por hora con `| sort 0 _time`. El orden de los códigos en una tabla de recuento no es el orden en que ocurrieron.

## Hashes: cómo usarlos

El campo `Hashes` de un evento de creación de proceso contiene varios valores en una sola cadena, por ejemplo `SHA1=...,MD5=...,SHA256=...,IMPHASH=...`. Para contrastarlos en fuentes externas como VirusTotal, **debe buscarse cada hash por separado**, no la cadena completa ni varios juntos.

## Pendiente

Para saber qué otros Event IDs de Sysmon existen realmente en el dataset, se consulta con `stats count by EventCode` (ver [`windows-event-ids.md`](windows-event-ids.md)) y se documentan solo los que aparezcan y se usen.
