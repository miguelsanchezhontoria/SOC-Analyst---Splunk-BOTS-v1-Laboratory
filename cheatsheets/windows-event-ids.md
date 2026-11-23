# Cheatsheet de Windows Event IDs

Chuleta de los identificadores de eventos nativos de Windows. Se rellena a medida que aparecen en el laboratorio.

> **Estado:** el escenario 1 (defacement web) se resolvió solo con eventos de **Sysmon**, así que todavía no hay IDs nativos con "visto en". Esta tabla se completará, previsiblemente, con el escenario 2.

## Sysmon frente a eventos nativos de Windows

Durante el escenario 1 se trabajó con registros Sysmon almacenados en el origen `XmlWinEventLog`. Los códigos identificados fueron:

| ID | Evento |
|---:|---|
| 1 | Process Creation |
| 3 | Network Connection |
| 5 | Process Terminated |
| 7 | Image Loaded |

Estos identificadores pertenecen a **Sysmon**, no a los registros nativos *Security*, *System* o *Application*. **Los mismos números pueden significar cosas distintas en cada canal**, así que conviene confirmar siempre de qué fuente procede el evento antes de interpretarlo. Los detalles de Sysmon están en [`sysmon.md`](sysmon.md).

## IDs nativos documentados

| ID | Canal | Qué significa | Por qué importa a un analista | Visto en |
|---:|---|---|---|---|
| — | — | Pendiente | Pendiente | — |

## Referencia general (aún sin verificar en el dataset)

Equivalencias habituales entre eventos nativos y de Sysmon. Se mantienen aquí hasta comprobar si el dataset los contiene.

| ID nativo | Canal | Qué significa | Equivalente aproximado en Sysmon |
|---:|---|---|---|
| 4624 | Security | Inicio de sesión correcto | — |
| 4625 | Security | Inicio de sesión fallido | — |
| 4688 | Security | Creación de un proceso (requiere auditoría activada) | ID 1 |

## Cómo saber qué IDs existen realmente

En lugar de documentar de memoria, se consultan los IDs presentes en el dataset:

```spl
index=botsv1 earliest=0 sourcetype=WinEventLog:Security
| stats count by EventCode
| sort 0 EventCode
```

Y los de Sysmon:

```spl
index=botsv1 earliest=0 sourcetype=XmlWinEventLog
| stats count by EventCode
| sort 0 EventCode
```

Para comprobar qué orígenes hay dentro de ese sourcetype:

```spl
index=botsv1 earliest=0 sourcetype=XmlWinEventLog
| stats count by source
```
