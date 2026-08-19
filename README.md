# Home-SOC-Lab-Laboratorio-de-Ciberseguridad-Blue-Team

Este proyecto forma parte de mi formación como estudiante de la **Tecnicatura en Ciberseguridad (FACEN - UNCa)**, con enfoque en roles de Blue Team / SOC Analyst.
 
## Objetivos
 
- Construir un entorno de laboratorio virtualizado que replique la arquitectura básica de un SOC real.
- Practicar la instalación, configuración y administración de un SIEM open source (Wazuh).
- Desarrollar habilidades de detección de amenazas mediante la generación de eventos controlados y su posterior análisis.
- Familiarizarme con herramientas y flujos de trabajo utilizados en la industria (SSH remoto, gestión de redes virtuales, hardening básico de infraestructura).
## Arquitectura del laboratorio
 
El lab está virtualizado en **VirtualBox** sobre un host Windows, con las siguientes máquinas planificadas:
 
| Rol | Sistema Operativo | Función |
|---|---|---|
| SIEM | Ubuntu Server 24.04 LTS | Wazuh Manager + Indexer + Dashboard |
| Atacante | Kali Linux | Generación de tráfico malicioso / pentesting |
| Víctima / Endpoint | Windows 11 LTSC (próximo paso) | Generación de logs y eventos para el SIEM |
 
Las máquinas se conectan mediante una **red interna aislada (Host-Only Network)** para no exponer el laboratorio a la red doméstica, sumado a un adaptador NAT independiente para actualizaciones y descarga de paquetes.
 
### Especificaciones de hardware (host)
- CPU: Intel i5-10400
- RAM: 16GB (distribuidos entre host y VMs)
- Hypervisor: Oracle VirtualBox
## Progreso actual
 
**Completado:**
- Instalación y configuración de Kali Linux (máquina atacante)
- Instalación de Ubuntu Server 24.04 LTS
- Configuración de red dual (NAT + Host-Only) para acceso a internet y administración remota simultánea
- Configuración de acceso remoto vía SSH
- Instalación de Wazuh 4.9 (Manager + Indexer + Dashboard) en modalidad single-node
- Acceso funcional al Dashboard web de Wazuh vía HTTPS
**Pendiente:**
- Instalación de VM Windows 11 LTSC (endpoint víctima)
- Despliegue del Wazuh Agent en el endpoint Windows
- Instalación de Sysmon para logging avanzado
- Simulación de ataques desde Kali (escaneos, fuerza bruta) y análisis de alertas generadas en Wazuh
## Stack tecnológico
 
- **VirtualBox** — Virtualización
- **Ubuntu Server 24.04 LTS** — Sistema base del SIEM
- **Wazuh 4.9** — SIEM / XDR open source
- **Kali Linux** — Distribución de pentesting
- **SSH** — Administración remota
- **Netplan** — Configuración de red en Ubuntu
## Aprendizajes clave
 
- Diferencia entre modelos de recolección de logs con agente (instalación local, más control y datos enriquecidos) y sin agente (recolección vía syslog/API).
- Configuración de redes virtuales en VirtualBox: NAT vs. Host-Only, y combinación de ambas para resolver escenarios reales de conectividad.
- Resolución de problemas comunes de virtualización (conflictos de drivers USB, configuración de red post-instalación mediante Netplan/YAML).
- Diferencias prácticas entre SIEMs open source (Wazuh) y comerciales (Splunk), y su relevancia según el contexto (lab personal vs. entorno enterprise).
- Gestión de recursos de hardware en entornos multi-VM, ajustando RAM y prioridades según el ejercicio a realizar.
## Nota sobre el proceso de aprendizaje
 
Este proyecto fue desarrollado con la asistencia de **Claude (Anthropic)** como guía técnica durante todo el proceso: desde la planificación de la arquitectura del laboratorio hasta la resolución de errores puntuales de configuración (redes, permisos, instalación del SIEM). El objetivo de usar esta herramienta fue acelerar el aprendizaje práctico y entender el "por qué" de cada paso, no solo copiar comandos, cada decisión de configuración fue explicada y comprendida antes de ejecutarla. Todo el trabajo de ejecución, troubleshooting en tiempo real y toma de decisiones sobre el hardware disponible fue realizado por mí.
 
## Próximos pasos
 
1. Instalar la VM Windows 11 LTSC como endpoint víctima.
2. Desplegar el Wazuh Agent y Sysmon en el endpoint.
3. Ejecutar simulaciones de ataque desde Kali (escaneo de puertos, fuerza bruta, etc.) y documentar la detección y triage de las alertas en el Dashboard de Wazuh.
4. Incorporar reglas de detección personalizadas en Wazuh.
5. Explorar un SIEM comercial (Splunk Free / trial) como comparación práctica.
---
 
**Autor:** Nicolás Emanuel Maldonado
**Formación:** Tecnicatura en Ciberseguridad — FACEN, UNCa (Sede Belén, Catamarca)
**Enfoque:** Blue Team / SOC Analyst
 
