# Home SOC Lab — Laboratorio de Ciberseguridad Blue Team
 
## Descripción del proyecto
 
Laboratorio casero de ciberseguridad orientado a la práctica como **Analista SOC (Security Operations Center)**, construido desde cero con herramientas open source. El objetivo es simular un entorno corporativo real donde se generan, recolectan y analizan eventos de seguridad, aplicando conceptos de detección de amenazas, gestión de logs y correlación de eventos mediante un SIEM.
 
Este proyecto forma parte de mi formación como estudiante de la **Tecnicatura en Ciberseguridad (FACEN - UNCa)**, con enfoque en roles de Blue Team / SOC Analyst.
 
## Objetivos
 
- Construir un entorno de laboratorio virtualizado que replique la arquitectura básica de un SOC real.
- Practicar la instalación, configuración y administración de un SIEM open source (Wazuh).
- Desarrollar habilidades de detección de amenazas mediante la generación de eventos controlados y su posterior análisis.
- Familiarizarme con herramientas y flujos de trabajo utilizados en la industria (SSH remoto, gestión de redes virtuales, hardening básico de infraestructura, resolución de problemas de compatibilidad de sistema).
## Arquitectura del laboratorio
 
El lab está virtualizado en **VirtualBox** sobre un host Windows, con las siguientes máquinas:
 
| Rol | Sistema Operativo | Función | IP |
|---|---|---|---|
| SIEM | Ubuntu Server 24.04 LTS | Wazuh Manager + Indexer + Dashboard | 192.168.56.102 |
| Víctima / Endpoint | Windows 11 LTSC | Wazuh Agent instalado — genera logs y eventos monitoreados | 192.168.56.103 |
| Atacante | Kali Linux (entorno gráfico Xfce completo) | Generación de tráfico malicioso / pentesting | 192.168.56.104 |
 
Las tres máquinas se conectan mediante una **red interna aislada (Host-Only Network, 192.168.56.0/24)** para no exponer el laboratorio a la red doméstica, sumado a un adaptador NAT independiente en cada VM para actualizaciones y descarga de paquetes.
 
### Especificaciones de hardware (host)
- CPU: Intel i5-10400
- RAM: 16GB (distribuidos entre host y VMs — Ubuntu/Wazuh 6GB, Windows 4GB, Kali 2-3GB)
- Hypervisor: Oracle VirtualBox 7.2.16
---
 
## Instalación y configuración — proceso completo
 
### 1. Ubuntu Server 24.04 LTS + Wazuh (SIEM)
 
**Instalación base**
Se creó la VM con disco de 40GB y arranque desde la ISO oficial de Ubuntu Server LTS. Durante la instalación no se marcó la opción de instalar OpenSSH Server, lo que requirió instalarlo manualmente después.
 
**Desafío — Conflicto de driver USB al iniciar la VM**
VirtualBox arrojó el error `VERR_PDM_USB_NAME_CLASH` al intentar arrancar. Se resolvió deshabilitando el controlador USB de la VM, innecesario para un servidor administrado por SSH:
- Configuración → USB → deshabilitar controlador USB
**Desafío — Acceso SSH sin conectividad**
Tras instalar Ubuntu, la VM solo tenía adaptador NAT (con internet, pero sin acceso desde el host). Se cambió a un único adaptador Host-Only para probar SSH, lo cual dejó a la VM sin internet y bloqueó la instalación de `openssh-server` (`apt` no podía descargar paquetes). Solución final: **dos adaptadores de red simultáneos**:
- Adaptador 1: NAT (internet)
- Adaptador 2: Host-Only (acceso SSH desde el host)
Con ambos activos, se instaló OpenSSH:
```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
```
 
**Desafío — Interfaz Host-Only sin IP asignada**
La interfaz `enp0s8` (Host-Only) no traía `dhcp4` habilitado en la configuración de Netplan. Se editó el archivo de configuración para agregarlo:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```
Agregando `dhcp4: true` bajo la sección `enp0s8`, y aplicando:
```bash
sudo netplan apply
```
Con esto, `enp0s8` obtuvo la IP `192.168.56.102`, habilitando la conexión SSH desde el host:
```bash
ssh vboxuser@192.168.56.102
```
 
**Instalación de Wazuh**
Se instaló Wazuh 4.9 en modo single-node (Manager + Indexer + Dashboard) usando el script oficial:
```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```
 
**Desafío — Requisitos de hardware no cumplidos**
El instalador rechazó continuar por RAM insuficiente (la VM tenía 4GB asignados, insuficientes para el mínimo recomendado de Wazuh). Se apagó la VM y se subió la RAM a **6GB** desde Configuración → Sistema, tras lo cual la instalación completó sin problemas.
 
Al finalizar, el Dashboard quedó accesible vía navegador en `https://192.168.56.102`, con las credenciales `admin` + contraseña generada por el instalador.
 
---
 
### 2. Windows 11 LTSC (Endpoint víctima)
 
**Configuración previa de la VM**
Windows 11 requiere firmware UEFI y TPM 2.0 habilitados, configurados antes de iniciar la instalación:
- Sistema → Placa base → habilitar UEFI
- Seguridad → habilitar chip TPM 2.0 y Secure Boot
- Dos adaptadores de red (NAT + Host-Only), igual que en Ubuntu
**Desafío — Pantalla en negro persistente al iniciar la VM**
La VM arrancaba pero no mostraba ninguna imagen. La causa fue doble:
1. Faltaba agregar la unidad óptica con la ISO montada en Almacenamiento (solo estaba el disco duro virtual, sin unidad de CD).
2. El controlador gráfico por defecto no era compatible con UEFI en esta versión de VirtualBox.
Solución: se agregó manualmente la unidad óptica con la ISO (Almacenamiento → Añadir → Elegir disco existente), y se cambió el controlador gráfico:
- Configuración → Pantalla → Controlador gráfico → **VMSVGA**
**Desafío — Pantalla en negro tras cada reinicio automático de Windows**
Durante la instalación, cada vez que Windows se reiniciaba automáticamente (paso normal del instalador), la pantalla quedaba congelada en negro sin avanzar visualmente, aunque el proceso seguía corriendo por detrás (confirmado revisando que el log de VirtualBox seguía creciendo en líneas). La solución práctica fue forzar el apagado y reinicio de la VM desde VirtualBox (Máquina → Cerrar → Apagar, luego iniciar de nuevo) cada vez que esto ocurría — esto "reengancha" el renderizado gráfico sin dañar la instalación en curso.
 
**Instalación del Wazuh Agent**
Con la red confirmada (`ping 192.168.56.102` exitoso desde Windows, con la VM Ubuntu encendida), se descargó e instaló el agente:
```
https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.2-1.msi
```
Durante la instalación se configuró la IP del Manager (`192.168.56.102`) y la Auth Key, generada en el lado del Manager:
```bash
sudo /var/ossec/bin/manage_agents
# Opción A (Add) → nombre del agente → IP 192.168.56.103
# Opción E (Extract) → ID del agente → copiar la clave generada
```
 
**Desafío — Agente duplicado tras corregir la clave**
Un primer intento de registro (`ID 001`) quedó como "Never connected" por un error de transcripción de la clave. Al corregirla y reiniciar el servicio del agente, este se auto-registró generando un nuevo `ID 002`, que sí quedó conectado (`Active`). Se eliminó el registro duplicado sin uso:
```bash
sudo /var/ossec/bin/manage_agents
# Opción R (Remove) → ID 001
```
 
Verificación final:
```bash
sudo /var/ossec/bin/agent_control -l
```
Confirmando el agente `win11-victima` (ID 002) en estado **Active**, visible también en la sección "Agents" del Dashboard de Wazuh.
 
---
 
### 3. Kali Linux (Atacante)
 
**Instalación**
Se instaló mediante el modo "Graphical Install", con idioma del sistema en inglés (`en_US.UTF-8`, por no existir el locale exacto para inglés + Argentina) y teclado en "Spanish (Latin American)". En la selección de software se marcaron los paquetes **Default** (entorno de escritorio + herramientas base), **Top 10** (herramientas más usadas) y **Collection of tools** (colección completa), desmarcando GNOME y KDE Plasma para no cargar con múltiples entornos gráficos innecesarios.
 
**Desafío — Sistema instalado sin entorno gráfico**
Tras la instalación, el login mostraba solo una consola de texto (TTY) en vez de una pantalla gráfica. Diagnóstico: el paquete de escritorio no había quedado instalado pese a haber sido seleccionado (`LightDM` y componentes de Xfce ausentes). Se instaló manualmente el meta-paquete oficial:
```bash
sudo apt update
sudo apt install kali-desktop-xfce -y
sudo systemctl enable lightdm
sudo systemctl start lightdm
```
Con esto, el sistema arrancó correctamente al escritorio gráfico Xfce.
 
**Confirmación de red**
```bash
ip a
```
Confirmando IP `192.168.56.104` en la interfaz Host-Only, en la misma subred que Ubuntu y Windows.
 
**Desafío — Guest Additions no compilaban (desfase kernel/headers)**
Al instalar las Guest Additions (necesarias para portapapeles compartido con el host), el portapapeles no funcionaba. El diagnóstico, paso a paso:
1. `lsmod | grep vboxguest` mostraba el módulo cargado, pero `/dev/vboxguest` no existía — indicio de instalación incompleta.
2. El log de instalación (`/var/log/vboxadd-setup.log`) reveló la causa raíz: *"kernel headers not found for target kernel"* — el kernel corriendo (`6.19.14+kali-amd64`) no coincidía con los headers instalados.
3. Kali, al ser una distribución *rolling release*, había actualizado los headers a una versión de kernel (`7.0.12+kali-amd64`) más nueva que la que el sistema tenía efectivamente en ejecución.
Solución: actualizar el sistema completo para sincronizar kernel y headers, y reiniciar:
```bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```
Tras el reinicio, `uname -r` confirmó el nuevo kernel (`7.0.12+kali-amd64`), coincidente con los headers ya instalados. Se reinstalaron las Guest Additions:
```bash
cd /media/cdrom0
sudo sh ./VBoxLinuxAdditions.run
sudo reboot
```
Verificación final:
```bash
ls -l /dev/vboxguest
```
El dispositivo existía correctamente. Además, se habilitó el portapapeles bidireccional en la configuración de la VM (Configuración → General → Avanzado → Portapapeles compartido → Bidireccional), quedando el copiar/pegar entre el host y Kali completamente funcional.
 
---
 
## Estado actual del laboratorio
 
**Completado:**
- Las 3 VMs instaladas, configuradas en red y verificadas end-to-end
- Wazuh Manager + Indexer + Dashboard funcionando en Ubuntu Server
- Wazuh Agent instalado, registrado y **Active** en el endpoint Windows
- Kali Linux con entorno gráfico completo y Guest Additions operativas
- Conectividad confirmada entre las 3 máquinas dentro de la red interna aislada
**Pendiente (próxima etapa):**
- Instalación de Sysmon en el endpoint Windows para logging avanzado
- Ejercicios de ataque desde Kali (escaneo de puertos con Nmap, fuerza bruta contra el login de Windows) y análisis de las alertas generadas en el Dashboard de Wazuh
- Incorporación de reglas de detección personalizadas
- Comparación práctica con un SIEM comercial (Splunk Free / trial)
## Stack tecnológico
 
- **VirtualBox 7.2.16** — Virtualización
- **Ubuntu Server 24.04 LTS** — Sistema base del SIEM
- **Wazuh 4.9** — SIEM / XDR open source (Manager + Agent)
- **Kali Linux** — Distribución de pentesting
- **Windows 11 LTSC** — Endpoint monitoreado
- **SSH** — Administración remota
- **Netplan** — Configuración de red en Ubuntu
## Aprendizajes clave
 
- Diferencia entre modelos de recolección de logs con agente (instalación local, más control y datos enriquecidos) y sin agente (recolección vía syslog/API).
- Configuración de redes virtuales en VirtualBox: NAT vs. Host-Only, y combinación de ambas en múltiples VMs para resolver escenarios reales de conectividad.
- Diagnóstico sistemático de fallas: aislar la causa raíz revisando logs (`vboxadd-setup.log`, `VBox.log`), comparando estados esperados vs. reales (`lsmod`, `ls /dev/`, `uname -r`), en vez de reintentar a ciegas.
- Requisitos y configuración de firmware UEFI/TPM 2.0 para instalar Windows 11 en un entorno virtualizado.
- Registro y autenticación de agentes Wazuh: diferencia entre registro manual (`manage_agents`, con IP fija) y auto-registro vía Auth Key, y por qué la identificación real del agente depende de la clave y no de la IP.
- Manejo de una distribución *rolling release* (Kali): comprensión del desfase entre kernel en ejecución y paquetes de headers disponibles en el repositorio, y cómo sincronizarlos con `apt full-upgrade`.
- Diferencias prácticas entre SIEMs open source (Wazuh) y comerciales (Splunk), y su relevancia según el contexto (lab personal vs. entorno enterprise).
- Gestión de recursos de hardware en entornos multi-VM, ajustando RAM y prioridades según el ejercicio a realizar.
## Nota sobre el proceso de aprendizaje
 
Quiero ser transparente sobre cómo trabajé en este proyecto: utilicé **Claude (Anthropic)** como asistente de IA durante todo el proceso, como guía técnica y tutor para entender conceptos y comandos que no conocía. Esto incluyó desde la planificación de la arquitectura del laboratorio hasta el diagnóstico y resolución de errores puntuales de configuración (redes, permisos, requisitos de UEFI/TPM, registro de agentes en Wazuh, y el desfase de kernel/headers en Kali).
 
Elegí trabajar así porque mi objetivo no era simplemente copiar comandos, sino **entender el "por qué" de cada paso** — qué hace cada configuración, por qué falla algo y cómo diagnosticarlo, y qué rol cumple cada componente dentro de un SIEM real. Cada decisión de configuración fue explicada, y solo la ejecuté una vez que la comprendí. Todo el trabajo de ejecución práctica, la resolución de errores en tiempo real (varios no triviales, como el problema de renderizado gráfico en VirtualBox con UEFI o el desfase kernel/headers en Kali), y las decisiones sobre el hardware y la arquitectura del lab fueron realizadas por mí.
 
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
