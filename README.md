# SOC-Analyst---Splunk-BOTS-v1-Laboratory
Portfolio - Ciberseguridad - Blue Team: Desarrollo de un laboratorio práctico de análisis de ciberseguridad utilizando Splunk Enterprise y el dataset Boss of the SOC (BOTS v1).

## Presentación
Splunk BOTS v1 es un conjunto de datos de seguridad realista y de código abierto creado por Splunk para entrenar a profesionales, investigadores y estudiantes en la búsqueda de amenazas (threat hunting) y análisis de incidentes. Este laboratorio tiene como finalidad el adquirir experiencia en seguridad defensiva usando una herramienta real.

Las habilidades adquiridas con este laboratorio son: SPL - Search Processing Language, lenguaje de consulta propio de Splunk -, análisis de logs de Windows, Sysmon, Suricata y tráfico de red, reconstrucción de incidentes con Kill Chain, mapeo a MITRE ATT&CK, escritura de detecciones y dashboards.

## Objetivos
Configuración del entorno según la guía oficial de Splunk BOTS v1 disponible en el repositorio de GitHub, resolver los escenarios de BOTS v1, reconstruir cada ataque con su timeline, escribir detecciones propias a partir de lo encontrado y documentar el proceso con dashboards.

## Organización del repositorio
├── README.md -> este archivo\
├── LICENSE -> explicado más adelante\
├── LICENSE-docs -> explicado más adelante\
├── botsv1 -> carpeta principal del repositorio\
│   ├── README.md -> resumen del escenario y timeline del ataque\
│   ├── writeups -> documentación detallada paso a paso de cómo se han resuelto los problemas planteados\
│   ├── spl.pdf -> documento con las consultas utilizadas durante todo el laboratorio\
│   ├── detections -> reglas/alertas descubiertas\
│   └── dashboards -> paneles visuales en formato XML/JSON\
├── cheatsheets -> chuletas sobre SPL, Windows Event IDs y Sysmon\
└── lab-setup -> carpeta con el documento en el que se explica la instalación y configuración del laboratorio\

## Licencias
- **Código** -> licencia MIT, ver LICENSE.
- **Documentación y write-ups** -> licencia CC BY 4.0, ver LICENSE-docs. Puedes reutilizarlos citando al autor.
- **Datasets BOTS**: propiedad de Splunk, publicados con licencia CC0. No se incluyen en este repositorio.
- Este proyecto no está afiliado ni respaldado por Splunk Inc.
