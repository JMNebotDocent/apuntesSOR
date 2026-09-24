# Sistemas Operativos en Red - Configuración del sistema de virtualización
*2º CFGM Sistemas Microinformáticos y Redes (SMR)*

---

## Tabla de Contenidos
1. [Introducción a la virtualización](#1-introducci%C3%B3n-a-la-virtualizaci%C3%B3n)
2. [Tipos de virtualización](#2-tipos-de-virtualizaci%C3%B3n)
3. [Ventajas e inconvenientes de la virtualización](#3-ventajas-e-inconvenientes-de-la-virtualizaci%C3%B3n)
    - [3.1. Ventajas](#31-ventajas)
    - [3.2. Inconvenientes](#32-inconvenientes)
4. [VirtualBox](#4-virtualbox)
    - [4.1. Antes de empezar](#41-antes-de-empezar)
    - [4.2. Descarga e instalación](#42-descarga-e-instalaci%C3%B3n)
    - [4.3. Interfaz del hipervisor](#43-interfaz-del-hipervisor)
    - [4.4. El botón Configuración](#44-el-bot%C3%B3n-configuraci%C3%B3n)
5. [Ejemplo de creación de una máquina virtual](#5-ejemplo-de-creaci%C3%B3n-de-una-m%C3%A1quina-virtual)
    - [5.1. Creación de la MV](#51-creaci%C3%B3n-de-la-mv)
    - [5.2. Instalación del Sistema Operativo Windows 10 en la MV](#52-instalaci%C3%B3n-del-sistema-operativo-windows-10-en-la-mv)
    - [5.3. Instalación de las Guest Additions](#53-instalaci%C3%B3n-de-las-guest-additions)
    - [5.4. Creación de una carpeta compartida host-guest](#54-creaci%C3%B3n-de-una-carpeta-compartida-host-guest)
    - [5.5. Creación de instantáneas](#55-creaci%C3%B3n-de-instant%C3%A1neas)
6. [Consejos finales](#6-consejos-finales)
7. [Resolución de problemas típicos](#7-resoluci%C3%B3n-de-problemas-t%C3%ADpicos)
8. [Anexos](#8-anexos)
    - [8.1. Resumen de la configuración de las máquinas virtuales Windows a crear](#81-resumen-de-la-configuraci%C3%B3n-de-las-m%C3%A1quinas-virtuales-windows-a-crear)
    - [8.2. Software de virtualización](#82-software-de-virtualizaci%C3%B3n)
9. [Bibliografía](#9-bibliograf%C3%ADa)

---

## 1. Introducción a la virtualización

De una manera bastante simple, podemos definir la virtualización como un software que permite simular un ordenador dentro de otro ordenador. Este equipo 'simulado' puede ejecutar su sistema operativo sobre el hardware real, aunque dependiendo del tipo de virtualización, la interacción con el hardware puede realizarse de una manera más transparente, o con más capas intermedias.

De ahora en adelante, utilizaremos la siguiente nomenclatura:

- **Hipervisor:** software que aprovecha y gestiona los recursos del sistema real (o anfitrión) para crear equipos simulados (máquinas virtuales).
- **Máquina virtual:** equipo virtual (o simulado) completamente funcional que consta de sistema operativo, acceso a red, dispositivos de almacenamiento, etc.
- **Equipo anfitrión, real o host:** equipo físico sobre el que se simulan otros equipos.
- **Equipo invitado, virtual o guest:** equipo simulado sobre el sistema real.

Las máquinas virtuales a las que se hará referencia son las llamadas **máquinas virtuales de sistema**, las cuales simulan o virtualizan un sistema completo. El otro gran grupo de máquinas virtuales son las llamadas **de proceso** (middleware), las cuales proporcionan un entorno de ejecución independiente de la plataforma hardware y del sistema operativo, como pueden ser Java o Microsoft .NET, pero que quedan fuera de los objetivos de este curso.

---

## 2. Tipos de virtualización

Podemos dividir en dos grandes grupos los esquemas de virtualización de sistema:

### Tipo I (Baremetal)
El hipervisor se halla incrustado en un sistema operativo muy ligero de manera que los recursos físicos del sistema real son aprovechados en casi su totalidad por los sistemas virtualizados.
> **Nota de rendimiento:** Según Proxmox, el rendimiento de los recursos hardware que se pierde al virtualizar con su sistema es inferior al 3% del que se obtendría al instalar directamente el sistema virtualizado sobre el hardware físico.

*Ejemplos:* Proxmox, Hyper-V y VMware ESXi.

### Tipo II
El hipervisor es un programa más ejecutándose dentro del sistema operativo instalado (Windows XP, Vista, 7, Ubuntu, openSUSE, Fedora, etc.) sobre la máquina real. Sobre este hipervisor se crean y ejecutan las máquinas virtuales.

*Ejemplos:* VirtualBox, VMware (Player, Workstation, etc.), QEMU, etc.

> [!NOTE]
> En los sistemas empresariales en los que se persigue un alto rendimiento y un elevado nivel de fiabilidad y disponibilidad, se implementan soluciones de **Tipo I**, ya que el objetivo es tener en funcionamiento sistemas servidores sobre una plataforma hardware accesibles y configurables a través de una consola o de la red.

---

## 3. Ventajas e inconvenientes de la virtualización

### 3.1. Ventajas

- **Ejecución simultánea:** Permite ejecutar diferentes sistemas operativos simultáneamente sobre un único hardware.
- **Instantáneas (Snapshots):** Permite crear estados definidos de la máquina pudiendo volver a ellos en caso de que alguna modificación haya causado daño en el sistema guest.
- **Ahorro de costes:** En entornos de producción con hardware potente, se aprovecha la capacidad del equipo reduciendo servidores físicos dedicados (ej. servidores de correo, aplicaciones y almacenamiento virtualizados en una sola máquina física al 90% de uso).
- **Aislamiento y seguridad:** Las aplicaciones ejecutadas en el SO guest se hallan aisladas del SO host. Ante un ataque por virus o malware, el sistema real está a salvo y se puede recuperar la MV desde una instantánea 'sana'.
- **Portabilidad y Alta Disponibilidad:** Los sistemas virtualizados pueden ser 'portados' a otro equipo físico de forma sencilla. En entornos de alta disponibilidad se crean clústers de virtualización para migrar MVs ante fallos de hardware.

### 3.2. Inconvenientes

- **Complejidad añadida:** Existencia de capas intermedias hasta llegar al hardware.
- **Pérdida de prestaciones:** Reducción del rendimiento ocasionada por las capas intermedias y la compartición de recursos hardware (aunque los esquemas *baremetal* reducen esta pérdida al mínimo).

---

## 4. VirtualBox

### 4.1. Antes de empezar

En clase utilizaremos VirtualBox como herramienta para virtualizar por varios motivos:
- Necesitamos virtualizar un sistema entero.
- Es una herramienta potente con todas las funcionalidades necesarias.
- Multiplataforma (Windows, GNU/Linux, macOS).
- Herramienta gratuita y válida para entornos educativos.
- Respaldo importante por parte de Oracle con actualizaciones frecuentes.

### 4.2. Descarga e instalación

1. Acceder a la web oficial: [virtualbox.org](https://www.virtualbox.org)
2. Descargar la versión adecuada según la arquitectura de nuestro equipo (32 o 64 bits).
3. Instalar el **Extension Pack** para la integración adecuada de USBs, arranque por red, etc. Existe una única versión del Extension Pack para todas las plataformas (requiere privilegios de administrador).

### 4.3. Interfaz del hipervisor

#### Menú Archivo:
- `Preferencias`: abre el menú con opciones generales de la aplicación.
- `Importar servicio virtualizado`: permite importar máquinas en formato OVF.
- `Exportar servicio virtualizado`: permite exportar una máquina virtual.
- `Herramientas -> Administrador de medios virtuales`: gestión de discos duros, unidades ópticas, disquetes.
- `Herramientas -> Administrador de red`: creación de redes NAT propias.
- `Herramientas -> Administrador de perfil cloud`: exportación a Oracle Cloud Infrastructure.
- `Comprobar actualizaciones`: verificar nuevas versiones del programa o Extension Pack.
- `Reiniciar todas las advertencias`: vuelve a chequear avisos o errores de configuración.

#### Menú Máquina:
- `Nueva...`: abre el asistente de creación de una nueva MV.
- `Añadir...`: añade una MV existente desde el disco.

### 4.4. El botón Configuración

Permite personalizar el hardware virtual de la máquina seleccionada:

- **General:** Nombre de la máquina y tipo/versión de SO a instalar.
- **Sistema:**
  - *Placa base:* Asignación de memoria RAM y orden de arranque.
  - *Procesador:* Configuración del número de núcleos (CPUs) asignados.
- **Almacenamiento:** Creación y gestión de controladores (SATA, IDE, SCSI, SAS), discos duros virtuales y unidades ópticas.
- **Red:** Configuración de los adaptadores de red habilitados.
- **Carpetas compartidas:** Configuración de carpetas compartidas entre host y guest.

---

## 5. Ejemplo de creación de una máquina virtual

### 5.1. Creación de la MV

1. Pulsar en el botón **'Nueva'**.
2. Indicar **Nombre** y **Tipo/Versión** de sistema operativo (se recomienda no rellenar el campo "Imagen ISO" en este paso para evitar una instalación desatendida automática).
3. Asignar **Memoria RAM** (mínimo recomendado: 512 MB).
4. Crear un **Disco duro virtual**:
   - Seleccionar **Reservado dinámicamente** (recomendado): el disco solo ocupará en el host el espacio real utilizado por la MV hasta el límite máximo fijado.
5. Finalizar el asistente e insertar la imagen ISO en la unidad óptica virtual desde la sección de *Almacenamiento*.

### 5.2. Instalación del Sistema Operativo Windows 10 en la MV

1. Iniciar la máquina virtual con el botón **'Iniciar'**.
2. Seleccionar idioma, formato de hora y teclado. Pulsar **'Instalar ahora'**.
3. En la clave de producto, seleccionar **"No tengo clave de producto"**.
4. Elegir la edición de Windows 10 deseada (ej. Windows 10 Pro) y aceptar la licencia.
5. Tipo de instalación: Elegir **"Personalizada: instalar solo Windows (avanzado)"**.
6. Selección de disco: Seleccionar el espacio sin asignar y continuar.
7. Al finalizar la copia de archivos y reiniciar:
   - Configurar región y teclado.
   - En la pantalla de inicio de sesión de Microsoft, elegir **"Cuenta sin conexión"** (o "Experiencia limitada").
   - Crear el usuario local (ej. `usuario`) y contraseña.
   - Configurar las preguntas de seguridad y opciones de privacidad.

> [!IMPORTANT]
> **Anotad el usuario y la contraseña** que hayáis asignado al sistema, ya que a lo largo del curso necesitaremos disponer de un usuario administrador local del equipo cliente en caso de problemas con el dominio.

### 5.3. Instalación de las Guest Additions

Las **Guest Additions** son un conjunto de controladores y aplicaciones que optimizan el rendimiento y la usabilidad de la MV (resolución adaptable, soporte USB, carpetas compartidas, portapapeles bidireccional).

#### Pasos para la instalación:
1. En el menú superior de la ventana de la MV: `Dispositivos` → `Instalar imagen de CD de Guest Additions...`.
2. Abrir el *Explorador de archivos* en Windows guest y acceder a la unidad de CD montada.
3. Ejecutar el instalador correspondiente (ej. `VBoxWindowsAdditions-amd64.exe`).
4. Seguir el asistente y **reiniciar la máquina virtual**.

### 5.4. Creación de una carpeta compartida host-guest

1. Menú `Dispositivos` → `Carpetas compartidas` → `Preferencias de carpetas compartidas...`.
2. Añadir nueva carpeta compartida:
   - **Ruta de la carpeta:** Seleccionar la carpeta en el equipo anfitrión.
   - **Nombre de la carpeta:** Identificador (ej. `SOR`).
   - Marcar las casillas **'Automontar'** y **'Hacer permanente'**.
3. Acceso desde la MV:
   - **Opción A:** Reiniciar la MV y comprobar la nueva unidad en *Este equipo*.
   - **Opción B (vía Consola CMD como Administrador):**
     ```cmd
     net use E: \\vboxsrv\SOR
     ```
     *(donde `E:` es la letra deseada y `SOR` el nombre dado a la carpeta).*

### 5.5. Creación de instantáneas

Permiten guardar el estado de la MV para restaurarlo posteriormente si ocurre algún fallo.

1. Con la máquina seleccionada o en marcha, hacer clic en el botón **'Tomar'** (sección Instantáneas).
2. Asignar un **Nombre** (ej. *Instantánea 1*) y **Descripción**.
3. Para restaurar: Apagar la MV, ir al administrador de instantáneas, seleccionar la instantánea guardada y hacer clic en **'Restaurar'**.

---

## 6. Consejos finales

- **Instala las Guest Additions** inmediatamente después de instalar el SO.
- **Configura la red atentamente:**
  - `NAT`: Otorga Internet a la MV, pero no permite comunicación con otras MVs ni equipos de la red local.
  - `Adaptador puente (Bridged)`: La MV se conecta al switch como un equipo real de la red física.
  - `Red interna`: Permite comunicación exclusivamente entre MVs del mismo host configuradas con el mismo nombre de red interna.
  - `Red NAT`: Permite crear subredes virtuales entre MVs con salida a Internet.
  - `Solo-anfitrión (Host-Only)`: Comunicación exclusiva entre la MV y el equipo anfitrión.
- **Haz copias de seguridad mediante Instantáneas** antes de realizar configuraciones delicadas en el SO.

---

## 7. Resolución de problemas típicos

> [!WARNING]
> - **Error al importar un archivo `.ova` (error de controlador USB):** Deshabilitar el controlador USB en las opciones del servicio virtualizado antes de importar.
> - **Error al intentar instalar desde la ISO:** Comprobar que se ha seleccionado la arquitectura correcta (32 bits vs 64 bits) al crear la MV en VirtualBox.

---

## 8. Anexos

### 8.1. Resumen de la configuración de las máquinas virtuales Windows a crear

| Sistema Operativo | RAM | Disco Duro | Red |
| :--- | :---: | :---: | :---: |
| **Windows Server 2016... 2022** | 1.5 - 2 GB | 25-32 GB (expansión dinámica) | 1 tarjeta de red ("Red NAT") |
| **Windows 10 (64 bits)** | 2 GB | 15 GB (expansión dinámica) | 1 tarjeta de red ("Red NAT") |
| **Windows 10 (32 bits)** | 1 GB | 15 GB (expansión dinámica) | 1 tarjeta de red ("Red NAT") |
| **Ubuntu 24.04 Server** | 1 - 4 GB | 25-30 GB (expansión dinámica) | 1 tarjeta de red ("Red NAT") |
| **Ubuntu 24.04 Desktop** | 1 - 4 GB | 25-30 GB (expansión dinámica) | 1 tarjeta de red ("Red NAT") |

### 8.2. Software de virtualización

#### VMware
- **Escritorio:** VMware Workstation (Windows/Linux, comercial), VMware Fusion (macOS Intel), VMware Player (gratuito para uso personal).
- **Empresarial (Tipo 1):** VMware ESXi (baremetal), vCenter (gestión centralizada con vMotion, Storage vMotion, DRS y HA).

#### Windows Server Hyper-V
Solución de virtualización de Microsoft integrada en Windows Server y Windows 10/11 Pro. Disponible como rol o como hipervisor independiente (*Hyper-V Server*).

#### Oracle VM VirtualBox
Hipervisor de Tipo 2 de código abierto (GPLv2) con Extension Pack bajo licencia PUEL. Excelente opción multiplataforma para entornos de escritorio y educativos.

#### Parallels Desktop for Mac
Optimizado para entornos Apple macOS, permitiendo ejecutar Windows y Linux con alto rendimiento.

#### Otras soluciones:
- **Windows Virtual PC:** Solución clásica de Microsoft para Windows 7 (virtualización de Windows XP).
- **Xen:** Hipervisor baremetal de código abierto que utiliza *paravirtualización* (penalización de rendimiento reducida al 2-8%).
- **OpenVZ:** Virtualización basada en contenedores a nivel de sistema operativo para GNU/Linux.

---

## 9. Bibliografía

- [SomeBooks.es - Sistemas Operativos en Red](http://somebooks.es/?p=4787)
- José Ramón Ruiz Rodríguez (2015). *Curso Cefire Windows 2008 Server*.
- José Ramón Ruiz Rodríguez (2015). *Curso Cefire Windows Server 2012*.
- [Wikipedia - Sistema Operativo de Red](http://es.wikipedia.org/wiki/Sistema_operativo_de_red)
- Elaboración propia (2024).
