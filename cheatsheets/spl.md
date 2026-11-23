# Cheatsheet de SPL

Chuleta de los comandos de Splunk (SPL) utilizados en el laboratorio. Solo se incluye lo que se ha usado de verdad; cada entrada indica dónde apareció.

> **Cómo se usa:** una búsqueda tiene una parte de filtro (antes del primer `|`) y una cadena de comandos separados por `|`, donde cada comando recibe el resultado del anterior.
>
> **Convención:** `<IP>`, `<campo>` y similares son marcadores que hay que sustituir.
>
> **Dónde se usaron:** "Esc. 1" remite a [`writeup1-Defacement_Web.pdf`](../botsv1/writeups/writeup1-Defacement_Web.pdf), seguido de la sección.

## 1. Filtros (parte inicial de la búsqueda)

| Elemento | Qué hace | Ejemplo |
|---|---|---|
| `index=botsv1` | Limita la búsqueda al índice del laboratorio | `index=botsv1 ...` | 
| `earliest=0` | Busca desde el instante 0 (1970), es decir, sin límite de fecha. Prevalece sobre el selector de tiempo de la pantalla. Necesario porque los eventos son de 2016 | `index=botsv1 earliest=0` |
| `sourcetype=<X>` | Selecciona los registros de un tipo de fuente | `sourcetype=stream:http` | 
| `src_ip="<IP>"` / `dest_ip="<IP>"` | Limita por IP de origen o destino | `src_ip="40.80.148.42"` | 
| `uri_path="<ruta>"` | Filtra por la ruta solicitada | `uri_path="/joomla/administrator/index.php"` | 
| `http_method=<M>` | Filtra por método HTTP. `POST` es el que se usa para enviar formularios y credenciales | `http_method=POST` | 
| `cookie="<valor>"` | Filtra por la cookie de sesión | `cookie="<cookie>"` | 
| `flow_id=<N>` | Selecciona los eventos del mismo flujo de red (Suricata) | `flow_id=<N>` | 
| `EventCode=<N>` | Selecciona un tipo de evento de Windows o Sysmon. Ver [`sysmon.md`](sysmon.md) | `EventCode=1` | 
| `ProcessGuid="{...}"` | Identifica un proceso concreto y permite correlacionar todos sus eventos | `ProcessGuid="{<GUID>}"` |
| Texto libre | Busca el término en todo el evento, sin indicar campo. Las comillas buscan una cadena exacta | `imreallynotbatman.com`, `".exe"`, `"3791.exe"` | 
| `*` | Comodín dentro de un valor | `form_data="*install_package*"` | 
| `OR` y paréntesis | Combina condiciones; los paréntesis agrupan | `(src_ip="<IP1>" OR src_ip="<IP2>")` | 

## 2. Comandos de transformación (después del `|`)

| Comando | Qué hace | Ejemplo | 
|---|---|---|
| `stats count by <campo>` | Cuenta los eventos agrupados por el valor de un campo. **Descarta los eventos que no tienen ese campo** | `stats count by src_ip` | 
| `stats count by <c1> <c2>` | Agrupa por la combinación de dos campos | `stats count by uri_path status` | 
| `sort - count` | Ordena de mayor a menor número de eventos | `sort - count` |
| `sort 0 _time` | Ordena cronológicamente **sin límite de resultados** (ver apartado 4) | `sort 0 _time` | 
| `sort EventCode` | Ordena por código de evento de menor a mayor | `sort EventCode` | 
| `table <campos>` | Muestra solo las columnas indicadas | `table _time src_ip src_headers` | 
| `dedup <campos>` | Conserva un solo evento por cada combinación de valores | `dedup site dest_ip` | 
| `search <condición>` | Filtra el resultado intermedio. Con `!=` se excluyen valores | `search status!="303"` | 
| `eval <campo>=<expr>` | Crea o modifica un campo calculado | `eval status=coalesce(status,"SIN_STATUS")` |
| `coalesce(a,b)` | Devuelve el primer valor que no sea nulo. Sirve para dar una etiqueta a eventos sin campo | `coalesce(status,"SIN_STATUS")` | 
| `if(isnull(x), a, b)` | Condicional: si el campo no existe, devuelve `a`; si no, `b` | `if(isnull(src_ip),"(sin src_ip)",src_ip)` | 
| `rex field=<campo> "<regex>"` | Extrae un nuevo campo con una expresión regular | Ver siguiente apartado | 

### Extraer un parámetro de un formulario con `rex`

```spl
| rex field=form_data "(?:^|&)passwd=(?<password>[^&]+)"
```

- `field=form_data`: campo del que se extrae.
- `(?:^|&)`: el parámetro empieza al principio del texto o tras un `&`.
- `passwd=`: nombre del parámetro buscado.
- `(?<password>[^&]+)`: captura, en un campo llamado `password`, todo lo que sigue hasta el siguiente `&`.

## 3. Búsquedas de administración del laboratorio

| Consulta | Qué hace |
|---|---|
| `\| rest /services/apps/local splunk_server=local \| table title label version visible disabled \| sort title` | Lista todas las apps instaladas, incluidas las que no aparecen en el menú, con su versión |
| `\| rest /services/apps/local splunk_server=local \| search disabled=0 \| table label version visible \| sort label` | Lista las apps que Splunk tiene habilitadas |
| `index=botsv1 earliest=0` | Muestra todos los eventos del índice, uno a uno |
| `index=botsv1 earliest=0 \| stats count` | Cuenta el total de eventos del índice |

## 4. Sourcetypes del laboratorio

| Sourcetype | Contenido |
|---|---|
| `stream:http` | Tráfico HTTP capturado en red |
| `suricata` | Alertas y eventos del IDS |
| `iis` | Registros del servidor web IIS |
| `fortigate_utm` | Registros de seguridad del firewall |
| `XmlWinEventLog` | Eventos de Windows en XML (Sysmon, en este laboratorio) |

## 5. Pendiente de documentar si se usan

`head`, `tail`, `timechart`, `streamstats`, `where`, `strftime`, `earliest()` y `latest()` dentro de `stats`.
