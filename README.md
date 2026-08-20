# Home SOC Lab — Laboratorio de Ciberseguridad Blue Team

## Descripción del proyecto

Laboratorio casero de ciberseguridad orientado a la práctica como **Analista SOC (Security Operations Center)**, construido desde cero con herramientas open source. El objetivo es simular un entorno corporativo real donde se generan, recolectan y analizan eventos de seguridad, aplicando conceptos de detección de amenazas, gestión de logs y correlación de eventos mediante un SIEM.

Este proyecto forma parte de mi formación como estudiante de la **Tecnicatura en Ciberseguridad (FACEN - UNCa)**, con enfoque en roles de Blue Team / SOC Analyst.

## Objetivos

- Construir un entorno de laboratorio virtualizado que replique la arquitectura básica de un SOC real.
- Practicar la instalación, configuración y administración de un SIEM open source (Wazuh).
- Desarrollar habilidades de detección de amenazas mediante la generación de eventos controlados y su posterior análisis.
- Familiarizarme con herramientas y flujos de trabajo utilizados en la industria (SSH remoto, gestión de redes virtuales, hardening básico de infraestructura).

## Arquitectura del laboratorio

El lab está virtualizado en **VirtualBox** sobre un host Windows, con las siguientes máquinas:

| Rol | Sistema Operativo | Función |
|---|---|---|
| SIEM | Ubuntu Server 24.04 LTS | Wazuh Manager + Indexer + Dashboard |
| Atacante | Kali Linux | Generación de tráfico malicioso / pentesting |
| Víctima / Endpoint | Windows 11 LTSC | Wazuh Agent instalado — genera logs y eventos monitoreados |

Las tres máquinas se conectan mediante una **red interna aislada (Host-Only Network)** para no exponer el laboratorio a la red doméstica, sumado a un adaptador NAT independiente en cada VM para actualizaciones y descarga de paquetes.

### Especificaciones de hardware (host)
- CPU: Intel i5-10400
- RAM: 16GB (distribuidos entre host y VMs)
- Hypervisor: Oracle VirtualBox

## Progreso actual

**Completado:**
- Instalación y configuración de Kali Linux (máquina atacante)
- Instalación de Ubuntu Server 24.04 LTS
- Configuración de red dual (NAT + Host-Only) en todas las VMs para acceso a internet y administración remota simultánea
- Configuración de acceso remoto vía SSH
- Instalación de Wazuh 4.9 (Manager + Indexer + Dashboard) en modalidad single-node
- Acceso funcional al Dashboard web de Wazuh vía HTTPS
- Instalación de Windows 11 LTSC como endpoint víctima, con UEFI, TPM 2.0 y Secure Boot habilitados
- Instalación y registro del Wazuh Agent en el endpoint Windows
- **Verificación end-to-end: el agente aparece Activo y conectado en el Dashboard de Wazuh**

**Pendiente:**
- Instalación de Sysmon para logging avanzado en el endpoint
- Simulación de ataques desde Kali (escaneo de puertos con Nmap, fuerza bruta) y análisis de las alertas generadas en Wazuh
- Incorporación de reglas de detección personalizadas

## Stack tecnológico

- **VirtualBox** — Virtualización
- **Ubuntu Server 24.04 LTS** — Sistema base del SIEM
- **Wazuh 4.9** — SIEM / XDR open source (Manager + Agent)
- **Kali Linux** — Distribución de pentesting
- **Windows 11 LTSC** — Endpoint monitoreado
- **SSH** — Administración remota
- **Netplan** — Configuración de red en Ubuntu

## Aprendizajes clave

- Diferencia entre modelos de recolección de logs con agente (instalación local, más control y datos enriquecidos) y sin agente (recolección vía syslog/API).
- Configuración de redes virtuales en VirtualBox: NAT vs. Host-Only, y combinación de ambas en múltiples VMs para resolver escenarios reales de conectividad.
- Resolución de problemas comunes de virtualización: conflictos de drivers USB, configuración de red post-instalación mediante Netplan/YAML, requisitos de firmware UEFI/TPM 2.0 para Windows 11, y comportamiento del controlador gráfico (VMSVGA) al reiniciar VMs con UEFI.
- Registro y autenticación de agentes Wazuh: diferencia entre registro manual (`manage_agents`, con IP fija) y auto-registro vía Auth Key, y por qué la identificación real del agente depende de la clave y no de la IP.
- Diagnóstico de conectividad de red entre VMs usando `ping` y `agent_control -l`, y resolución de problemas de firewall/estado de máquinas apagadas.
- Diferencias prácticas entre SIEMs open source (Wazuh) y comerciales (Splunk), y su relevancia según el contexto (lab personal vs. entorno enterprise).
- Gestión de recursos de hardware en entornos multi-VM, ajustando RAM y prioridades según el ejercicio a realizar.

## Nota sobre el proceso de aprendizaje

Quiero ser transparente sobre cómo trabajé en este proyecto: utilicé **Claude (Anthropic)** como asistente de IA durante todo el proceso, como guía técnica y tutor para entender conceptos y comandos que no conocía. Esto incluyó desde la planificación de la arquitectura del laboratorio hasta la resolución de errores puntuales de configuración (redes, permisos, requisitos de UEFI/TPM para Windows 11, registro de agentes en Wazuh, entre otros).

Elegí trabajar así porque mi objetivo no era simplemente copiar comandos, sino **entender el "por qué" de cada paso** — qué hace cada configuración, por qué falla algo y cómo diagnosticarlo, y qué rol cumple cada componente dentro de un SIEM real. Cada decisión de configuración fue explicada, y solo la ejecuté una vez que la comprendí. Todo el trabajo de ejecución práctica, la resolución de errores en tiempo real (varios no triviales, como problemas de renderizado gráfico en VirtualBox con UEFI o duplicación de agentes en Wazuh), y las decisiones sobre el hardware y la arquitectura del lab fueron realizadas por mí.

Considero que usar herramientas de IA de forma criteriosa, como complemento del estudio y no como reemplazo de la comprensión, es parte de las habilidades que un analista SOC necesita hoy — investigar, entender documentación técnica, y resolver problemas con las herramientas disponibles.

## Próximos pasos

1. Instalar Sysmon en el endpoint Windows para logging avanzado (procesos, conexiones de red).
2. Ejecutar simulaciones de ataque desde Kali (escaneo de puertos con Nmap, fuerza bruta contra el login de Windows) y documentar la detección y triage de las alertas en el Dashboard de Wazuh.
3. Incorporar reglas de detección personalizadas en Wazuh.
4. Explorar un SIEM comercial (Splunk Free / trial) como comparación práctica.

---

**Autor:** Nicolás Emanuel Maldonado
**Formación:** Tecnicatura en Ciberseguridad — FACEN, UNCa (Sede Belén, Catamarca)
**Enfoque:** Blue Team / SOC Analyst
