# UD04 — Configuración de Windows

**Módulo:** Sistemas Operativos Monopuesto · 1º SMR

## Introducción

En UD03 aprendiste a convivir con tu Ubuntu: arrancarlo, elegir su entorno, montar discos, recuperarlo cuando no arranca, instalar software y automatizar tareas. En esta unidad vas a hacer **exactamente lo mismo, pero en Windows 11**, sobre la VM `Apellido-UD02-Windows` que instalaste en UD02.

Las preguntas son idénticas a las de UD03 (¿cómo arranca?, ¿dónde se monta un disco?, ¿qué hago si no arranca?, ¿cómo instalo y actualizo?, ¿cómo programo una tarea?), pero las respuestas cambian. Por eso no vamos a empezar de cero: cada vez que un concepto ya lo viste en Linux, lo recordaremos en una línea y nos centraremos en **cómo lo resuelve Windows y en qué se diferencia**. En el trabajo te vas a encontrar los dos sistemas, a menudo en la misma oficina, y lo que se espera de un técnico es que sepa moverse en ambos con la misma soltura.

Al terminar esta unidad debes ser capaz de:

- Explicar el proceso de arranque de Windows, entender qué es el inicio rápido y acceder a las opciones avanzadas de arranque; gestionar sesiones de usuario.
- Usar con criterio la interfaz gráfica, el Símbolo del sistema (CMD) y PowerShell, y personalizar el entorno de trabajo.
- Preparar un disco nuevo en Windows, elegir su sistema de archivos (NTFS, exFAT, FAT32, ReFS) y montarlo con letra de unidad o en una carpeta.
- Aplicar las herramientas de recuperación de Windows: puntos de restauración, `sfc` y `DISM`, el Entorno de recuperación (WinRE) y el restablecimiento del equipo.
- Mantener Windows actualizado con Windows Update y entender los distintos tipos de actualización.
- Instalar y desinstalar software con instaladores clásicos, con Microsoft Store y con `winget`.
- Configurar la red y los dispositivos con los asistentes del sistema y desde la terminal.
- Automatizar tareas con el Programador de tareas y con scripts de PowerShell.

**Todas las prácticas se hacen en la VM, no en el equipo del aula**, aunque el equipo del aula también tenga Windows 11. Vas a forzar el modo seguro, tocar discos y particiones, restaurar el sistema y provocar averías, y en una VM todo eso se deshace en un minuto. En el equipo real, un error puede dejarlo sin arrancar, y además casi todo requiere permisos de administrador que tu usuario del aula no tiene.

**Antes de empezar:** toma una instantánea de tu VM Windows con el nombre "Inicio UD04". En esta unidad vas a romper cosas a propósito, y esa instantánea es tu red de seguridad.

**Cómo está pensada esta unidad:** igual que en UD03, cada comando o herramienta nueva viene con un cuadro **"Vamos a practicar"**: primero una guía **paso a paso** y, después, una pregunta para **reflexionar**. Haz siempre la parte guiada antes de pasar a la reflexión.

> **Edición de Windows.** Estos apuntes están escritos para **Windows 11 Pro**, que es la edición que usas en clase y la habitual en las empresas. Cuando algo no exista en Windows 11 Home, se indica.

---

## 1. Arranque y sesiones

### Qué pasa entre que pulsas el botón y ves tu escritorio

En UD03 viste el arranque de Linux: firmware → GRUB → núcleo → systemd → sesión. Windows sigue **la misma idea en las mismas fases**, aunque con piezas que se llaman distinto:

| Fase | Linux (Ubuntu) | Windows 11 |
|---|---|---|
| Firmware | UEFI | UEFI |
| Gestor de arranque | GRUB (`grubx64.efi`) | **Administrador de arranque de Windows** (`bootmgfw.efi`), que lee su configuración del **BCD** |
| Cargador del sistema | GRUB carga el núcleo | **`winload.efi`** carga el núcleo y los controladores imprescindibles |
| Núcleo | `vmlinuz` | **`ntoskrnl.exe`** |
| Primer proceso de usuario | `systemd` (PID 1) | **`smss.exe`** (Administrador de sesiones), que lanza el resto |
| Servicios | Unidades `.service` de systemd | **`services.exe`** arranca los servicios de Windows |
| Pantalla de inicio de sesión | Gestor de pantalla (GDM) | **`winlogon.exe`** y **`LogonUI.exe`** |
| Escritorio | Entorno de escritorio (GNOME) | **`explorer.exe`** |

El **BCD** (*Boot Configuration Data*, datos de configuración de arranque) es el equivalente al archivo de configuración de GRUB: una pequeña base de datos guardada en la partición EFI que dice qué sistemas se pueden arrancar y con qué opciones. No se edita con un editor de texto, sino con el comando `bcdedit`, que verás más adelante.

No tienes que memorizar los nombres de todos los procesos. Lo importante es que entiendas que **el arranque es una cadena**, que cada eslabón carga al siguiente y que, si uno falla, el síntoma que ves depende de en qué eslabón se ha roto: no es lo mismo un error antes del logotipo de Windows (firmware o gestor de arranque) que un sistema que llega a la pantalla de inicio de sesión pero se cuelga al entrar (sesión de usuario o algún programa de inicio).

### "Apagar" no siempre apaga: el inicio rápido

Aquí está la diferencia de arranque entre Windows y Linux que más problemas da en el día a día de un técnico. Windows 11 trae activado por defecto, en la mayoría de los equipos, el **inicio rápido** (*Fast Startup*).

Con el inicio rápido activado, cuando pulsas **Apagar**, Windows cierra tu sesión y tus programas, pero **no apaga del todo el núcleo**: guarda el estado del núcleo y de los controladores en el disco (en el archivo `C:\hiberfil.sys`, el mismo que usa la hibernación) y, al encender, lo recupera en lugar de arrancar desde cero. Es una "hibernación a medias", y por eso el equipo enciende más rápido.

El problema es que el núcleo **nunca se reinicia de verdad** al apagar y encender. Y eso tiene consecuencias muy prácticas:

- Un controlador que se ha quedado en mal estado sigue en mal estado al día siguiente, aunque el usuario "haya apagado el ordenador".
- Algunas actualizaciones y cambios de hardware no se aplican hasta un **reinicio**.
- En un equipo con **arranque dual**, el disco de Windows queda marcado como "en uso", y Linux solo puede montarlo en modo lectura para no corromperlo.

Fíjate en la paradoja: en Windows, con el inicio rápido activado, **Reiniciar hace un arranque más completo que Apagar y encender**. Por eso la primera pregunta de cualquier soporte técnico, "¿has probado a reiniciar?", tiene sentido aquí: reiniciar, no apagar y encender.

Si necesitas un apagado completo de verdad tienes tres opciones:
- mantener pulsada la tecla `Mayús` mientras haces clic en **Apagar**;
- ejecutar `shutdown /s /t 0` en una terminal (el modificador `/s` hace un apagado completo; solo con `/hybrid` usaría el inicio rápido);
- desactivar el inicio rápido de forma permanente, en **Panel de control → Hardware y sonido → Opciones de energía → Elegir el comportamiento de los botones de inicio/apagado → Cambiar la configuración actualmente no disponible**.

### Las opciones avanzadas de arranque

Windows no muestra un menú de arranque visible como GRUB: para que el arranque sea rápido, va directo al sistema. Si necesitas entrar en el menú de **opciones avanzadas** (para arrancar en modo seguro, restaurar el sistema, abrir una terminal de reparación...), tienes varias formas de llegar:

- Mantener pulsada `Mayús` mientras haces clic en **Reiniciar**.
- **Configuración → Sistema → Recuperación → Inicio avanzado → Reiniciar ahora**.
- Desde una terminal: `shutdown /r /o /t 0` (reiniciar, `/o` = ir a las opciones avanzadas).
- Si Windows **falla al arrancar dos veces seguidas**, a la tercera entra solo en este menú (la "Reparación automática").

Este menú es en realidad un pequeño sistema operativo de reparación llamado **WinRE** (*Windows Recovery Environment*, Entorno de recuperación de Windows), que verás a fondo en el apartado 3. Desde él, la ruta **Solucionar problemas → Opciones avanzadas → Configuración de inicio → Reiniciar** te lleva a una lista numerada en la que eliges, por ejemplo, **4) Habilitar modo seguro** o **5) Habilitar modo seguro con funciones de red**.

El **modo seguro** arranca Windows con lo mínimo imprescindible: controladores básicos, sin la mayoría de servicios y sin los programas que se inician solos con la sesión. Es el equivalente aproximado al `multi-user.target` de Linux en un sentido concreto: un arranque reducido para diagnosticar. Si un problema **desaparece** en modo seguro, ya sabes que la causa está en algo que no se carga en ese modo (un controlador, un servicio o un programa de inicio). Si **persiste**, la causa es más profunda.

### Sesiones de usuario

Igual que en Linux, una **sesión** es el periodo en el que un usuario autenticado tiene procesos asociados a su cuenta, y puede haber **varias sesiones a la vez**. Las cuatro acciones que en UD03 te pedimos no confundir tienen su equivalente directo, y en Windows aparecen todas juntas al pulsar `Ctrl + Alt + Supr`:

| Acción | Qué ocurre | Atajo o comando |
|---|---|---|
| **Bloquear** | La sesión sigue abierta tal cual; solo pide la contraseña para volver | `Win + L` |
| **Cambiar de usuario** | Tu sesión **sigue abierta** en segundo plano y otro usuario abre la suya | Desde `Ctrl + Alt + Supr` o el menú Inicio |
| **Cerrar sesión** | Se cierran tus programas y vuelves a la pantalla de inicio de sesión | `shutdown /l` o `logoff` |
| **Apagar / Reiniciar** | Se cierran todas las sesiones de todos los usuarios | `shutdown /s /t 0` · `shutdown /r /t 0` |

"Cambiar de usuario" es la opción que más sorprende: el primer usuario no ha cerrado nada, así que sus programas siguen ocupando memoria. Si el segundo usuario pulsa **Apagar**, Windows avisa de que hay otra persona con la sesión iniciada y de que puede perder lo que no haya guardado.

El comando `quser` (o `query user`) muestra las sesiones abiertas en el equipo, con su estado: **Activo** para la que está en pantalla, **Desconectado** para la que sigue abierta en segundo plano. Es el equivalente a `loginctl list-sessions`. (No existe en Windows 11 Home.)

> **Vamos a practicar: dos sesiones a la vez**
>
> Para esta práctica necesitas un segundo usuario. La gestión de usuarios la estudiarás a fondo en UD06; aquí solo vamos a crear uno de prueba.
>
> **Paso a paso:**
>
> 1. **Configuración → Cuentas → Otros usuarios → Agregar cuenta**. Pulsa "No tengo los datos de inicio de sesión de esta persona" y después "Agregar un usuario sin cuenta Microsoft". Llámalo `prueba`, con una contraseña sencilla y las tres preguntas de seguridad.
> 2. Abre el Bloc de notas, escribe algo y **no lo guardes**.
> 3. Pulsa `Ctrl + Alt + Supr` → **Cambiar de usuario** y entra como `prueba`. La primera vez tardará un poco: Windows está creando su perfil.
> 4. Abre una terminal (clic derecho en el botón Inicio → **Terminal**) y ejecuta `quser`. Anota cuántas sesiones aparecen y en qué estado está cada una.
> 5. Ejecuta `whoami` para confirmar con qué usuario estás trabajando.
> 6. Desde el menú Inicio, intenta **Apagar** el equipo. Lee el aviso que aparece y **cancela**.
> 7. Cierra la sesión de `prueba` (`shutdown /l`) y vuelve a entrar con tu usuario: el Bloc de notas sigue ahí, con el texto sin guardar.
>
> **Reflexiona:** un compañero de trabajo usa "Cambiar de usuario" en un equipo compartido en lugar de cerrar sesión, porque "es más rápido". ¿Qué dos problemas puede causar esa costumbre: uno de rendimiento y otro de pérdida de datos?

> **Vamos a practicar: comprueba el inicio rápido y entra en las opciones avanzadas**
>
> **Paso a paso:**
>
> 1. Abre **Panel de control → Hardware y sonido → Opciones de energía → Elegir el comportamiento de los botones de inicio/apagado**. Anota si la casilla **Activar inicio rápido** aparece, y si está marcada. (En algunas VMs no aparece porque la hibernación está desactivada; si es tu caso, anótalo también: es un resultado válido.)
> 2. Abre una terminal y ejecuta `shutdown /r /o /t 0`. La VM se reiniciará directamente en el menú azul de opciones avanzadas.
> 3. Recorre **Solucionar problemas → Opciones avanzadas** y anota las herramientas que ofrece. No ejecutes ninguna todavía.
> 4. Entra en **Configuración de inicio → Reiniciar** y, en la lista numerada, pulsa `4` para arrancar en **modo seguro**.
> 5. Inicia sesión. Fíjate en la resolución de pantalla, en el fondo negro y en el texto "Modo seguro" de las esquinas. Abre el Administrador de tareas (`Ctrl + Mayús + Esc`) y compara a ojo cuántos procesos hay con los que ves en un arranque normal.
> 6. Reinicia normalmente con `shutdown /r /t 0`.
>
> **Reflexiona:** un usuario te dice que su equipo "va lentísimo desde que instaló un programa", pero que en modo seguro va bien. ¿Qué te dice eso sobre la causa? ¿Y si en modo seguro también fuera lentísimo?

---

## 2. Interfaces de usuario y personalización

### Tres formas de hablar con Windows

Ya sabes por UD01 y UD03 que la interfaz gráfica (GUI) y la línea de comandos (CLI) no compiten, sino que se complementan. En Windows, la línea de comandos tiene además una particularidad: **conviven dos intérpretes distintos**.

- **Símbolo del sistema (CMD, `cmd.exe`)**: el intérprete heredado de MS-DOS. Sigue presente por compatibilidad: muchos scripts antiguos (`.bat`) y bastantes comandos clásicos (`ipconfig`, `ping`, `shutdown`, `sfc`, `chkdsk`) se siguen usando desde aquí todos los días. Pero Microsoft ya no le añade funciones nuevas.
- **PowerShell**: el intérprete moderno de Windows, y el que se usa hoy para administrar sistemas Windows de forma profesional. Windows 11 trae **Windows PowerShell 5.1**; existe además **PowerShell 7**, multiplataforma (funciona incluso en Linux y macOS), que se instala aparte.
- **Terminal Windows** (*Windows Terminal*): no es un intérprete, sino la **ventana** que los aloja. Es la aplicación de terminal por defecto en Windows 11 y permite tener pestañas de PowerShell, CMD e incluso Linux (si se instala WSL) a la vez.

Los comandos clásicos como `ipconfig` o `ping` son programas `.exe` independientes, así que funcionan igual desde CMD que desde PowerShell. Lo que cambia es todo lo demás.

### Qué hace distinto a PowerShell

Las órdenes propias de PowerShell se llaman **cmdlets** y siguen siempre el patrón **Verbo-Sustantivo**: `Get-Process` (obtener procesos), `Stop-Process` (detener un proceso), `Get-Service`, `Get-ChildItem` (listar el contenido de una carpeta), `New-Item`, `Remove-Item`... Ese patrón fijo tiene una ventaja enorme cuando estás aprendiendo: si sabes qué quieres hacer, puedes **adivinar** el nombre del cmdlet, y si no lo adivinas, puedes buscarlo:

- `Get-Command *service*`: lista todos los cmdlets que tienen "service" en el nombre.
- `Get-Help Get-Service -Examples`: muestra ejemplos de uso de un cmdlet.

La segunda diferencia es más profunda. En Linux (y en CMD), la tubería `|` pasa **texto** de un comando al siguiente, y por eso necesitas `grep`, `cut` o `awk` para recortar ese texto. En PowerShell, la tubería pasa **objetos**: cada proceso, cada archivo o cada servicio viaja con todas sus propiedades por separado (nombre, tamaño, fecha...). Así puedes ordenar, filtrar y elegir columnas por su nombre, sin recortar texto:

```powershell
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU
```

Esto se lee casi como una frase: "obtén los procesos, ordénalos por uso de CPU de mayor a menor y quédate con los 5 primeros, mostrando solo el nombre y la CPU".

Para que el salto desde Linux o CMD no sea brusco, PowerShell incluye **alias**: puedes escribir `ls`, `dir`, `cd`, `cat`, `cp`, `mv` o `rm` y funcionan, porque por debajo llaman a su cmdlet (`ls` y `dir` son alias de `Get-ChildItem`). Pero **solo se parecen**: `ls -l`, por ejemplo, da error, porque `Get-ChildItem` no tiene ese parámetro.

| Tarea | Linux (bash) | CMD | PowerShell |
|---|---|---|---|
| Listar una carpeta | `ls -l` | `dir` | `Get-ChildItem` (alias `ls`, `dir`) |
| Cambiar de carpeta | `cd` | `cd` | `Set-Location` (alias `cd`) |
| Ver un archivo de texto | `cat` | `type` | `Get-Content` (alias `cat`, `type`) |
| Buscar texto | `grep` | `findstr` | `Select-String` |
| Ver procesos | `ps` | `tasklist` | `Get-Process` |
| Ayuda de un comando | `man ls` | `dir /?` | `Get-Help Get-ChildItem` |
| Quién soy | `whoami` | `whoami` | `whoami` |
| Limpiar la pantalla | `clear` | `cls` | `Clear-Host` (alias `cls`, `clear`) |

Una precaución importante: algunas tareas necesitan **privilegios de administrador**, igual que en Linux necesitabas `sudo`. En Windows no se antepone nada al comando: hay que **abrir la terminal como administrador** (clic derecho en el botón Inicio → **Terminal (administrador)**) y aceptar el aviso del **Control de cuentas de usuario** (UAC). Se nota porque el título de la ventana empieza por "Administrador:". Si un comando te responde "Acceso denegado" o "requiere elevación", casi siempre es esto.

> **Vamos a practicar: primeros pasos en PowerShell**
>
> **Paso a paso, en Terminal Windows (pestaña de PowerShell):**
>
> 1. `$PSVersionTable.PSVersion`: comprueba qué versión de PowerShell tienes.
> 2. `Get-ChildItem C:\Windows`: lista la carpeta de Windows. Prueba después `ls C:\Windows` y `dir C:\Windows` y comprueba que el resultado es el mismo.
> 3. `ls -l C:\Windows`: lee el error. ¿Por qué falla, si `ls` sí funcionaba?
> 4. `Get-Command *service*`: busca los cmdlets relacionados con servicios. Fíjate en los verbos que aparecen (`Get`, `Start`, `Stop`, `Restart`...).
> 5. `Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU`: los cinco procesos que más CPU han consumido.
> 6. Abre una pestaña de **Símbolo del sistema** en la misma ventana (flecha ˅ junto a las pestañas) y ejecuta `ipconfig`. Vuelve a la pestaña de PowerShell y ejecuta también `ipconfig`: funciona igual en las dos.
> 7. En CMD, prueba `Get-Process`. ¿Qué ocurre y por qué?
>
> **Reflexiona:** en Linux, para obtener solo el nombre de los procesos de la salida de `ps aux` tendrías que recortar columnas de texto. En PowerShell has elegido la columna `Name` por su nombre. ¿Qué ventaja tiene eso si mañana cambia el orden o el ancho de las columnas que muestra el comando?

### Configuración y Panel de control: dos paneles para lo mismo

Windows 11 tiene **dos** herramientas gráficas de configuración que conviven: la aplicación **Configuración** (moderna, la que Microsoft quiere que uses) y el **Panel de control** (heredado de versiones anteriores). Microsoft lleva años pasando opciones del Panel de control a Configuración, pero la migración no está terminada: hay ajustes que **solo** están en uno de los dos, y otros que aparecen en ambos, a veces con nombres distintos.

Para un técnico, la consecuencia práctica es que conviene conocer los dos y saber abrirlos directamente. Muchas ventanas clásicas tienen un nombre corto que puedes escribir en el cuadro **Ejecutar** (`Win + R`) o en la terminal:

| Comando | Abre | Lo usarás en |
|---|---|---|
| `control` | Panel de control | Esta unidad |
| `ncpa.cpl` | Conexiones de red (adaptadores) | Apartado 5 |
| `appwiz.cpl` | Programas y características | Apartado 4 |
| `sysdm.cpl` | Propiedades del sistema (incluye Protección del sistema) | Apartado 3 |
| `diskmgmt.msc` | Administración de discos | Apartado 3 |
| `devmgmt.msc` | Administrador de dispositivos | Apartado 5 |
| `taskschd.msc` | Programador de tareas | Apartado 6 |
| `ms-settings:` | Aplicación Configuración | Esta unidad |

Los que acaban en `.msc` son **consolas de administración** (*Microsoft Management Console*), y verás muchas más en UD06.

### Personalizar el entorno

En Linux pudiste **cambiar de entorno de escritorio entero** (GNOME, XFCE, KDE Plasma...). En Windows eso no existe: hay un único entorno gráfico, que forma parte del propio sistema (el proceso `explorer.exe` dibuja el escritorio, la barra de tareas y el menú Inicio). Lo que sí puedes es personalizarlo a fondo desde **Configuración → Personalización**: fondo, colores y modo oscuro, temas, pantalla de bloqueo, menú Inicio y barra de tareas.

Hay además varios ajustes que, sin ser "decoración", cambian mucho la forma de trabajar y que un técnico configura casi siempre en un equipo nuevo:

- **Explorador de archivos → Ver → Mostrar**: activa **Extensiones de nombre de archivo** y **Elementos ocultos**. Ya viste en UD01 por qué ocultar las extensiones es un riesgo (el clásico `factura.pdf.exe`).
- **Escritorios virtuales** (`Win + Tab`): varios escritorios independientes en la misma sesión, por ejemplo uno para el trabajo y otro para documentación. `Win + Ctrl + D` crea uno nuevo y `Win + Ctrl + ←/→` cambia entre ellos.
- **Aplicaciones predeterminadas** (**Configuración → Aplicaciones → Aplicaciones predeterminadas**): con qué programa se abre cada tipo de archivo o enlace.
- **Accesibilidad**: tamaño del texto, contraste, lupa, narrador... Imprescindible cuando preparas el equipo de una persona con baja visión o cualquier otra necesidad.

> **Ojo, licencia sin activar.** Si tu VM Windows **no está activada** (en UD02 podías instalar sin clave de producto), la mayoría de opciones de **Personalización** aparecen en gris y no se pueden cambiar. No es un fallo de la VM: es una limitación deliberada de Windows sin activar, y un ejemplo real de lo que viste sobre licencias en UD02. El resto de la unidad funciona igual. Si es tu caso, documenta el mensaje que aparece y personaliza lo que sí se puede (el explorador, los escritorios virtuales, las aplicaciones predeterminadas y la accesibilidad).

> **Vamos a practicar: prepara el entorno como lo haría un técnico**
>
> **Paso a paso:**
>
> 1. **Explorador de archivos → Ver → Mostrar**: activa las extensiones y los elementos ocultos. Comprueba que ahora ves la carpeta `C:\ProgramData`, que antes no aparecía.
> 2. **Configuración → Personalización**: cambia el fondo, activa el modo oscuro y alinea la barra de tareas a la izquierda (**Barra de tareas → Comportamientos de la barra de tareas**). Si tu Windows no está activado, anota el mensaje que aparece.
> 3. Crea un segundo escritorio virtual con `Win + Ctrl + D`, abre en él el Bloc de notas y cambia entre escritorios con `Win + Ctrl + ←/→`.
> 4. **Configuración → Aplicaciones → Aplicaciones predeterminadas**: busca la extensión `.txt` y comprueba con qué programa se abre.
> 5. Pulsa `Win + R`, escribe `sysdm.cpl` y comprueba que se abre directamente la ventana clásica de Propiedades del sistema.
>
> **Reflexiona:** en Linux pudiste instalar un segundo entorno de escritorio completo y elegirlo al iniciar sesión. ¿Por qué crees que en Windows no es posible, teniendo en cuenta lo que viste en UD01 sobre quién desarrolla cada sistema y cómo se distribuye?

---

## 3. Sistemas de archivos y recuperación

### Los sistemas de archivos de Windows

En UD01 viste qué es un sistema de archivos y qué aporta el *journaling*. Estos son los que te vas a encontrar en un equipo Windows y cuándo tiene sentido cada uno:

| Sistema de archivos | Dónde se usa | Ventajas | Limitaciones |
|---|---|---|---|
| **NTFS** | Disco del sistema y discos internos | *Journaling*, permisos (UD06), compresión, cifrado, cuotas | Linux lo lee y escribe, pero macOS solo lo lee sin software extra |
| **exFAT** | Pendrives y discos externos grandes | Lo leen y escriben Windows, macOS, Linux, televisores, cámaras... | Sin *journaling* ni permisos: una extracción sin "Expulsar" puede corromperlo |
| **FAT32** | Pendrives pequeños, partición EFI | Compatible con prácticamente cualquier dispositivo | **Ningún archivo puede superar 4 GB**; sin *journaling* ni permisos |
| **ReFS** | Servidores; *Unidad de desarrollo* de Windows 11 | Diseñado para volúmenes enormes y para detectar y corregir datos dañados | No admitido como disco del sistema en Windows 11 |

El límite de 4 GB de FAT32 es una causa clásica de incidencia: "intento copiar una ISO de 5 GB a un pendrive con 32 GB libres y me dice que no hay espacio". Hay espacio de sobra; lo que no hay es forma de guardar un archivo tan grande en ese formato.

### Preparar un disco nuevo: inicializar, particionar, formatear y asignar letra

En Linux montabas el disco en una carpeta (`/mnt/datos`) y lo hacías permanente en `/etc/fstab`. En Windows la idea es la misma, pero la forma habitual de "montar" es asignar al volumen una **letra de unidad** (`D:`, `E:`...). Y aquí no hay ningún `fstab` que editar: **Windows recuerda la letra de cada volumen por sí solo**, guardándola en el registro asociada a un identificador único del volumen, de forma parecida a como en Linux usabas el UUID en lugar de `/dev/sdb1`.

La herramienta gráfica para todo esto es **Administración de discos** (`diskmgmt.msc`). El proceso para un disco recién conectado tiene cuatro pasos, y es el mismo que hiciste en UD03 con `fdisk`, `mkfs` y `mount`:

1. **Inicializar** el disco: elegir el esquema de particiones, **GPT** (el moderno, que ya conoces de UD02) o MBR.
2. **Crear un volumen** (una partición) en el espacio sin asignar.
3. **Formatearlo** con un sistema de archivos.
4. **Asignarle una letra** o montarlo en una carpeta.

Esta última opción es la que menos gente conoce: Windows también permite **montar un volumen en una carpeta vacía de un disco NTFS**, exactamente como en Linux. Por ejemplo, el disco nuevo puede aparecer como `C:\Datos` en lugar de como `E:`. Es útil cuando se acaban las letras o cuando quieres que un disco extra "amplíe" una carpeta concreta sin que el usuario note que es otro disco.

El equivalente en línea de comandos es **`diskpart`**, una herramienta interactiva en la que primero **seleccionas** un disco o volumen y después actúas sobre él:

```
diskpart
DISKPART> list disk            ← discos físicos, con número y tamaño
DISKPART> list volume          ← volúmenes, con letra, sistema de archivos y etiqueta
DISKPART> select volume 3      ← a partir de aquí, todo afecta al volumen 3
DISKPART> assign letter=R      ← asigna la letra R:
DISKPART> remove letter=R      ← quita la letra (el volumen sigue ahí, con sus datos)
DISKPART> exit
```

> ⚠️ **`diskpart` no pide confirmación.** Órdenes como `clean` (borra todo el disco seleccionado) se ejecutan al instante. Antes de cada acción, comprueba con `list disk` o `list volume` que el asterisco `*` marca el disco o volumen que crees. Equivocarse de número es el error más grave que se puede cometer con esta herramienta.

En PowerShell tienes cmdlets de solo consulta, que no cambian nada y que son una forma segura de ver qué hay: `Get-Disk`, `Get-Partition` y `Get-Volume`.

> **Vamos a practicar: prepara un segundo disco**
>
> **Preparación (con la VM apagada):** en VirtualBox, **Configuración → Almacenamiento**, añade al controlador SATA un **disco duro nuevo** de 2 GB (VDI, reservado dinámicamente), igual que hiciste en UD03 con la VM Linux.
>
> **Paso a paso:**
>
> 1. Arranca la VM, pulsa clic derecho en el botón Inicio → **Administración de discos**. Windows detectará el disco nuevo y te pedirá inicializarlo: elige **GPT**.
> 2. Clic derecho sobre el espacio **No asignado** del Disco 1 → **Nuevo volumen simple**. Usa todo el espacio, asigna la letra `D:` (o la primera libre), formato **NTFS** y etiqueta `DATOS`.
> 3. Abre el Explorador y crea en `DATOS` un archivo `prueba.txt` con algo escrito.
> 4. Abre una terminal **como administrador** y ejecuta `Get-Volume`. Localiza tu volumen `DATOS`: anota su letra, su sistema de archivos y su tamaño.
> 5. Crea una carpeta vacía `C:\Datos`. En Administración de discos, clic derecho sobre el volumen `DATOS` → **Cambiar la letra y rutas de acceso de unidad → Agregar → Montar en la siguiente carpeta NTFS vacía** → `C:\Datos`.
> 6. Abre `C:\Datos` en el Explorador: ves `prueba.txt`. El mismo volumen es ahora accesible por dos caminos, `D:\` y `C:\Datos`.
> 7. Reinicia la VM y comprueba que la letra y la carpeta siguen funcionando sin que hayas tenido que configurar nada más.
>
> **Reflexiona:** en UD03 tuviste que escribir una línea en `/etc/fstab` para que el disco se montara solo en cada arranque. ¿Por qué en Windows no ha hecho falta? ¿Dónde crees que ha guardado Windows esa información?

> **Vamos a practicar: la letra que "desaparece"**
>
> **Paso a paso, en una terminal como administrador:**
>
> 1. `diskpart` y después `list volume`. Localiza el número del volumen `DATOS`.
> 2. `select volume <número>` y comprueba con `list volume` que el asterisco señala el volumen correcto.
> 3. `remove letter=D` (o la letra que tenga). Sal con `exit`.
> 4. Abre el Explorador: `D:` ya no aparece en **Este equipo**. ¿Se han perdido los datos? Abre `C:\Datos`.
> 5. Vuelve a `diskpart`, selecciona el volumen y ejecuta `assign letter=D`. Comprueba que la unidad vuelve a aparecer con `prueba.txt` dentro.
>
> **Reflexiona:** un usuario llama porque "su disco de copias ha desaparecido" del Explorador, pero el disco se ve en Administración de discos, con su tamaño y su etiqueta. ¿Qué es lo primero que comprobarías? ¿Por qué no hay que formatearlo bajo ningún concepto?

### Cuando Windows no funciona bien: la escalera de recuperación

En Linux, ante un sistema que no arranca, entrabas por GRUB con `init=/bin/bash` o desde una ISO *live*. Windows tiene un abanico de herramientas más amplio, y la clave profesional es usarlas **en orden, de la menos drástica a la más drástica**, como si subieras una escalera. Cada peldaño resuelve más problemas que el anterior, pero también cambia más cosas:

| Peldaño | Herramienta | Qué hace | Qué se pierde |
|---|---|---|---|
| 1 | **Reiniciar** (no apagar) | Arranque completo del núcleo | Nada |
| 2 | **Modo seguro** | Arrancar con lo mínimo para diagnosticar y quitar lo que causa el fallo | Nada |
| 3 | **`sfc` y `DISM`** | Comprobar y reparar archivos del sistema dañados | Nada |
| 4 | **Restaurar sistema** | Devolver programas, controladores, registro y configuración a un punto anterior | Programas y controladores instalados después de ese punto |
| 5 | **Desinstalar la última actualización** (desde WinRE) | Quitar una actualización que ha roto el arranque | Esa actualización |
| 6 | **Restablecer este PC** | Reinstalar Windows | Todos los programas; los archivos personales, si eliges "Quitar todo" |
| 7 | **Reinstalación limpia** desde una ISO | Empezar de cero | Todo lo que no tenga copia de seguridad |

Fíjate en que **ningún peldaño, salvo el último, borra tus archivos personales** si eliges bien las opciones. Y en que ninguna de estas herramientas sustituye a una **copia de seguridad**: Restaurar sistema, por ejemplo, no guarda tus documentos, y si el disco se estropea no sirve de nada.

### Puntos de restauración

Un **punto de restauración** es una "foto" del estado del sistema (archivos de sistema, registro, controladores y programas instalados) que Windows puede recuperar más adelante. Es una idea muy parecida a las **instantáneas** de VirtualBox que usas desde UD02, con dos diferencias importantes: solo afecta a la parte de sistema, **no a tus documentos**, y la gestiona el propio Windows desde dentro.

Windows crea puntos automáticamente antes de instalar actualizaciones o controladores, **pero solo si la protección del sistema está activada**, y en muchas instalaciones de Windows 11 viene **desactivada** de serie. Se activa en `sysdm.cpl` → pestaña **Protección del sistema** → seleccionar la unidad `C:` → **Configurar** → **Activar protección del sistema**, reservando algo de espacio (un 5-10% del disco es razonable). Desde esa misma pestaña puedes crear un punto a mano con **Crear** y volver a uno anterior con **Restaurar sistema** (también se abre con `rstrui`).

### `sfc` y `DISM`: reparar archivos del sistema

Si Windows arranca pero se comporta de forma extraña (pantallazos azules ocasionales, herramientas del sistema que no se abren, errores al actualizar), una causa posible es que algún **archivo del propio sistema** esté dañado. Windows incluye dos herramientas que trabajan en equipo, y ambas se ejecutan desde una terminal **como administrador**:

- **`DISM /Online /Cleanup-Image /RestoreHealth`** repara el **almacén de componentes**: la copia de referencia de los archivos de Windows que el sistema guarda en `C:\Windows\WinSxS`. Si hace falta, descarga las piezas buenas de Windows Update.
- **`sfc /scannow`** (*System File Checker*) compara los archivos del sistema en uso con esa copia de referencia y sustituye los que no coinciden.

El orden importa, y la razón es lógica: `sfc` repara **usando** el almacén de componentes. Si ese almacén está dañado, `sfc` no puede reparar bien. Por eso se ejecuta **primero `DISM` y después `sfc`**. Tardan varios minutos cada uno, y el resultado aparece al final en una frase que conviene leer con atención: si no encontró problemas, si encontró y reparó, o si encontró problemas que no pudo reparar.

> **Vamos a practicar: activa la protección del sistema y usa un punto de restauración**
>
> **Paso a paso:**
>
> 1. `Win + R` → `sysdm.cpl` → pestaña **Protección del sistema**. Comprueba si la protección está activada para `C:`. Si no lo está, actívala con **Configurar** y reserva un 8% del disco.
> 2. Pulsa **Crear** y llama al punto `Antes de tocar el registro`.
> 3. Haz un cambio de sistema fácil de reconocer: en una terminal **como administrador**, ejecuta `reg add "HKLM\SOFTWARE\SOM-UD04" /v Prueba /t REG_SZ /d "antes del punto no existía"`. Comprueba que existe con `reg query "HKLM\SOFTWARE\SOM-UD04"`.
> 4. Crea además un archivo en tu **Escritorio** llamado `importante.txt`.
> 5. En **Protección del sistema**, pulsa **Restaurar sistema**, elige el punto `Antes de tocar el registro` y sigue el asistente. La VM se reiniciará y tardará unos minutos.
> 6. Al volver, ejecuta de nuevo `reg query "HKLM\SOFTWARE\SOM-UD04"`. Busca también `importante.txt` en tu Escritorio.
>
> **Reflexiona:** la clave del registro ha desaparecido, pero `importante.txt` sigue ahí. ¿Qué te dice esto sobre lo que protege un punto de restauración y lo que **no** protege? ¿En qué se diferencia de una instantánea de VirtualBox?

> **Vamos a practicar: comprueba la integridad del sistema**
>
> **Paso a paso, en una terminal como administrador:**
>
> 1. `DISM /Online /Cleanup-Image /RestoreHealth`: espera a que llegue al 100% (puede quedarse parado un buen rato en el 62% o en el 84%; es normal).
> 2. `sfc /scannow`: anota literalmente la frase final que muestra.
>
> **Reflexiona:** explica con tus palabras por qué se ejecuta `DISM` antes que `sfc` y no al revés.

### WinRE: el entorno de recuperación

Cuando Windows ni siquiera llega a arrancar, las herramientas anteriores (que se usan desde dentro de Windows) no sirven. Para eso está **WinRE**, el pequeño sistema de reparación que viste en el apartado 1. Además de entrar desde el propio Windows, se puede llegar a él de dos formas cuando el sistema está roto: **automáticamente**, tras dos arranques fallidos seguidos, o **arrancando desde la ISO o el USB de instalación** de Windows y eligiendo **Reparar el equipo** en lugar de Instalar.

En **Solucionar problemas → Opciones avanzadas** encontrarás:

- **Reparación de inicio**: intenta corregir sola los problemas más comunes de arranque. Es lo primero que se prueba.
- **Desinstalar actualizaciones**: quita la última actualización de calidad o de características. Muy útil cuando el equipo dejó de arrancar justo después de actualizar.
- **Restaurar sistema**: los mismos puntos de restauración, pero sin necesidad de que Windows arranque.
- **Configuración de inicio**: el modo seguro.
- **Símbolo del sistema**: una terminal con privilegios totales para reparar a mano.
- **Configuración de firmware UEFI**: reinicia directamente en la configuración UEFI del equipo.

Desde **Solucionar problemas** también aparece **Restablecer este PC**, que reinstala Windows y te deja elegir entre **Mantener mis archivos** y **Quitar todo**, y entre **Descarga en la nube** (descarga Windows de nuevo) o **Reinstalación local** (usa los archivos que ya hay en el disco).

### Reparar el arranque desde la terminal de WinRE

Si el problema es el propio gestor de arranque o el BCD (la situación equivalente a "GRUB está roto" en UD03), se repara desde el **Símbolo del sistema** de WinRE. En equipos UEFI, la herramienta correcta es **`bcdboot`**, que vuelve a copiar los archivos de arranque a la partición EFI y regenera el BCD:

```
diskpart
DISKPART> list volume                   ← localiza la partición EFI (FAT32, ~100 MB, sin letra)
DISKPART> select volume <número>
DISKPART> assign letter=S
DISKPART> exit
bcdboot C:\Windows /s S: /f UEFI
```

Dos trampas habituales en esta terminal:

- **La letra de Windows puede no ser `C:`**. WinRE asigna las letras a su manera, y la partición de Windows suele aparecer como `D:`. Antes de nada, compruébalo con `list volume` o con `dir D:\Windows`.
- **Muchos tutoriales antiguos recomiendan `bootrec /fixmbr`, `/fixboot` y `/rebuildbcd`**. Son herramientas pensadas para el arranque BIOS con MBR; en un equipo UEFI con GPT, `bootrec /fixboot` suele terminar con "Acceso denegado" y no arregla nada. Es un buen ejemplo de procedimiento que ha quedado anticuado.

Tu profesor o profesora te mostrará en directo la reparación con `bcdboot`, porque requiere romper antes el arranque de la VM.

### `bcdedit`: ver (y tocar) la configuración de arranque

`bcdedit` (en una terminal como administrador) muestra el contenido del BCD. Ejecutado sin parámetros, enseña el **Administrador de arranque de Windows** y una entrada **Cargador de arranque de Windows** por cada sistema instalado, identificada como `{current}` (la del sistema que está en marcha) o `{default}` (la que arranca por defecto).

También permite cambiar opciones del arranque. Un ejemplo útil: `bcdedit /set {current} safeboot minimal` hace que **todos** los arranques siguientes sean en modo seguro, hasta que se deshace con `bcdedit /deletevalue {current} safeboot`. Lo mismo se puede hacer gráficamente con `msconfig` → pestaña **Arranque** → **Arranque a prueba de errores**. Es útil cuando necesitas entrar varias veces seguidas en modo seguro, y también una fuente clásica de incidencias: alguien lo activa, se olvida, y el equipo "siempre arranca raro".

> ⚠️ Antes de modificar el BCD, haz una copia: `bcdedit /export C:\bcd_copia`. Si algo sale mal, se recupera con `bcdedit /import C:\bcd_copia`.

> **Vamos a practicar: explora WinRE y el BCD**
>
> **Paso a paso:**
>
> 1. En una terminal como administrador, ejecuta `bcdedit` y localiza la entrada `{current}`. Anota el valor de `device`, `path` y `description`.
> 2. `bcdedit /export C:\bcd_copia` para guardar una copia del BCD.
> 3. `shutdown /r /o /t 0` para entrar en WinRE. Ve a **Solucionar problemas → Opciones avanzadas → Símbolo del sistema** (te pedirá elegir tu cuenta y su contraseña).
> 4. En esa terminal, ejecuta `dir C:\` y `dir D:\`. ¿En qué letra está realmente tu carpeta `Windows` dentro de WinRE?
> 5. Ejecuta `diskpart` y `list volume`. Identifica la partición EFI (FAT32, unos 100 MB) y la partición de recuperación. Sal con `exit` sin cambiar nada.
> 6. Escribe `exit` para cerrar la terminal y elige **Continuar** para volver a Windows.
>
> **Reflexiona:** si en el paso 4 hubieras dado por hecho que Windows estaba en `C:` y hubieras lanzado una reparación sobre esa letra, ¿qué podría haber pasado?

---

## 4. Gestión de software: actualización e instalación

### Windows Update: no todas las actualizaciones son iguales

En Ubuntu, `apt upgrade` actualizaba **todo** el software instalado desde los repositorios: el sistema y los programas. En Windows, **Windows Update** se encarga del sistema operativo y de algunos componentes de Microsoft, pero **no** de la mayoría de programas de terceros (los navegadores, 7-Zip o VLC tienen su propio sistema de actualización o se actualizan con `winget`, como verás enseguida).

Windows Update distribuye varios tipos de actualización, y conviene distinguirlos porque se gestionan de forma distinta:

- **Actualizaciones de calidad** (*acumulativas*): llegan cada mes, el segundo martes (el famoso *Patch Tuesday*), con parches de seguridad y correcciones. Se llaman acumulativas porque cada una **incluye todas las anteriores**: un equipo que lleva meses sin actualizar solo necesita la última, no todas una a una.
- **Actualizaciones de características**: una vez al año, añaden funciones nuevas y cambian la versión de Windows (por ejemplo, de 24H2 a 25H2: el año y la mitad del año en que se publica). Cada versión tiene soporte durante un tiempo limitado, y cuando termina deja de recibir parches de seguridad.
- **Actualizaciones de controladores**, de **definiciones de Microsoft Defender** (varias veces al día) y **actualizaciones opcionales** (versiones de prueba de la siguiente acumulativa, controladores alternativos...).

Desde **Configuración → Windows Update** puedes, además de buscar e instalar:

- **Pausar las actualizaciones** durante un máximo de cinco semanas. Útil si sabes que una actualización concreta está dando problemas, peligroso si se olvida.
- **Opciones avanzadas → Horas activas**: el periodo en el que Windows no reiniciará el equipo por su cuenta para terminar de instalar.
- **Historial de actualizaciones**: lo que se ha instalado, con su código **KB** (*Knowledge Base*, el identificador de cada actualización), y desde ahí **Desinstalar actualizaciones**.

El código KB es lo que buscas en internet cuando una actualización da problemas: "KB5xxxxxx impresora no funciona" te llevará directamente a otros técnicos con el mismo problema y, a menudo, a la página de Microsoft que lo reconoce.

En una empresa, las actualizaciones no se suelen dejar en manos de cada equipo: se gestionan de forma centralizada para probarlas antes en unos pocos equipos y desplegarlas después en el resto. Lo verás en módulos de segundo curso.

> **Vamos a practicar: revisa el estado de actualización de tu Windows**
>
> **Paso a paso:**
>
> 1. **Configuración → Windows Update → Buscar actualizaciones**. Instala lo que haya pendiente.
> 2. Abre **Historial de actualizaciones** y anota el código KB de la última **actualización acumulativa** instalada.
> 3. `Win + R` → `winver`. Anota la **versión** (por ejemplo, 25H2) y la **compilación del SO**.
> 4. En **Opciones avanzadas**, configura las **horas activas** de 8:00 a 15:00.
> 5. Pausa las actualizaciones una semana y comprueba qué mensaje aparece en la página principal de Windows Update. Después, pulsa **Reanudar actualizaciones**.
>
> **Reflexiona:** ¿por qué crees que Microsoft limita a cinco semanas el tiempo máximo de pausa, en lugar de dejar que el usuario las pause indefinidamente?

### Tres formas de instalar programas

En Ubuntu, casi todo el software llegaba desde los **repositorios** con APT o Snap. En Windows, la tradición ha sido otra: **cada programa se descarga de la web de su fabricante** y trae su propio instalador. Hoy conviven tres formas de instalar:

| Forma | Qué es | Ventajas | Inconvenientes |
|---|---|---|---|
| **Instalador `.exe`** | Un programa que instala otro programa. Cada fabricante hace el suyo | Disponible para casi todo | Cada uno funciona distinto; hay que fiarse de dónde se descarga; algunos intentan colar software extra |
| **Paquete `.msi`** | Formato estándar de **Windows Installer** | Instalación y desinstalación estándar y limpias; se puede instalar sin preguntas (`msiexec /i paquete.msi /qn`) y desplegar en muchos equipos | Menos habitual en programas domésticos |
| **Microsoft Store** (MSIX) | La tienda oficial de Microsoft | Aplicaciones aisladas del resto del sistema, se actualizan solas y se desinstalan sin dejar restos | No todo el software está en la tienda |

Si te recuerda a lo que viste en UD03, es porque se parece mucho: las aplicaciones de la Store, aisladas y autocontenidas, son la idea equivalente a **Snap**.

Para **desinstalar**, la vía estándar es **Configuración → Aplicaciones → Aplicaciones instaladas** (o la clásica `appwiz.cpl`). Muchos desinstaladores dejan restos (configuración en `C:\Users\<usuario>\AppData` y entradas en el registro), igual que `apt remove` dejaba la configuración; pero en Windows no existe un `purge` universal: depende de lo bien hecho que esté cada desinstalador.

### `winget`: el gestor de paquetes de Windows

Durante años, instalar un programa en Windows ha significado buscarlo en internet, esquivar las webs de descarga falsas y hacer clic en "Siguiente" varias veces. **`winget`** (*Administrador de paquetes de Windows*), incluido en Windows 11, cambia eso: es una herramienta de línea de comandos que funciona **casi exactamente como APT**.

| Tarea | APT (Ubuntu) | winget (Windows) |
|---|---|---|
| Buscar un programa | `apt search vlc` | `winget search vlc` |
| Ver información | `apt show vlc` | `winget show --id VideoLAN.VLC` |
| Instalar | `sudo apt install vlc` | `winget install --id VideoLAN.VLC -e` |
| Ver lo instalado | `apt list --installed` | `winget list` |
| Ver actualizaciones pendientes | `apt list --upgradable` | `winget upgrade` |
| Actualizar todo | `sudo apt upgrade` | `winget upgrade --all` |
| Desinstalar | `sudo apt remove vlc` | `winget uninstall --id VideoLAN.VLC -e` |

Algunas cosas que debes saber:

- Cada paquete tiene un **identificador** único (`VideoLAN.VLC`, `7zip.7zip`...). Usar `--id` con `-e` (*exact*, coincidencia exacta) evita instalar por error otro programa con un nombre parecido.
- `winget` usa dos **orígenes**: su repositorio comunitario (`winget`) y la Microsoft Store (`msstore`). El repositorio comunitario no guarda los programas, sino una ficha con la dirección de descarga oficial del fabricante y una huella (*hash*) para comprobar que el archivo descargado es exactamente el esperado.
- La primera vez te pedirá aceptar las condiciones de los orígenes (o puedes añadir `--accept-source-agreements --accept-package-agreements`).
- **`winget upgrade --all` actualiza también programas que no instalaste con `winget`**, siempre que los reconozca. Es lo más parecido que hay en Windows a actualizar todo el software de un equipo con un único comando.

Y una función muy útil para preparar equipos: `winget export -o apps.json` guarda en un archivo la lista de programas instalados, y `winget import -i apps.json` los instala todos en otro equipo. Preparar las 30 máquinas de un aula con los mismos programas pasa de ser una mañana de clics a un comando por equipo.

> **Vamos a practicar: instala y desinstala de tres formas**
>
> **Paso a paso:**
>
> 1. **Microsoft Store**: busca e instala **Microsoft PowerToys** (un conjunto de utilidades de Microsoft para usuarios avanzados). Ábrelo y prueba **Selector de colores** con `Win + Mayús + C`.
> 2. En una terminal, `winget search 7zip` y localiza el identificador exacto del programa 7-Zip.
> 3. `winget install --id 7zip.7zip -e`: acepta lo que te pida (y el aviso de UAC). Comprueba en el menú Inicio que 7-Zip aparece.
> 4. `winget list --id 7zip.7zip`: comprueba que aparece instalado y con qué versión.
> 5. `winget upgrade`: anota qué programas tienen actualizaciones pendientes. Fíjate en si aparecen programas que tú no instalaste con `winget`.
> 6. `winget uninstall --id 7zip.7zip -e`. Comprueba que ya no aparece en **Configuración → Aplicaciones → Aplicaciones instaladas**.
> 7. Desinstala PowerToys desde **Configuración → Aplicaciones → Aplicaciones instaladas**.
>
> **Reflexiona:** en UD03 viste que añadir un PPA equivale a confiar en quien lo mantiene. ¿En quién confías cuando instalas un programa con `winget`? ¿Y cuando lo descargas desde el primer resultado de un buscador?

> **Vamos a practicar: prepara la lista de programas de un aula**
>
> 1. Instala con `winget` dos programas útiles para un aula de informática, por ejemplo `VideoLAN.VLC` y `Notepad++.Notepad++`.
> 2. `winget export -o C:\Users\Public\aula.json`: abre el archivo con el Bloc de notas y localiza los identificadores de los programas que has instalado.
> 3. Desinstala los dos programas con `winget uninstall`.
> 4. `winget import -i C:\Users\Public\aula.json --accept-package-agreements`: comprueba que se vuelven a instalar.
>
> **Reflexiona:** el archivo exportado incluye muchos más programas que los dos que instalaste. ¿De dónde salen? ¿Lo editarías antes de usarlo para preparar un aula? ¿Por qué?

---

## 5. Asistentes de configuración: red y dispositivos

### Configurar la red

Igual que en Ubuntu tenías el applet gráfico, `nmtui` y `nmcli` para hablar con NetworkManager, en Windows tienes un asistente gráfico y varias herramientas de terminal. Todas modifican la **misma configuración**, así que lo que cambias desde una se ve desde las demás.

- **Gráficamente**: **Configuración → Red e Internet → Ethernet** muestra la configuración de la conexión. En **Asignación de IP → Editar** puedes pasar de **Automático (DHCP)** a **Manual** e introducir IP, máscara (como longitud de prefijo, por ejemplo 24), puerta de enlace y DNS. La ventana clásica de **Conexiones de red** (`ncpa.cpl`) sigue disponible y permite hacer lo mismo desde las propiedades de **Protocolo de Internet versión 4 (TCP/IPv4)**.
- **`ipconfig`**: el comando clásico para consultar. `ipconfig` muestra lo básico y `ipconfig /all` muestra también la dirección MAC, si la IP viene por DHCP y qué servidores DNS se usan. `ipconfig /release` y `ipconfig /renew` liberan y piden de nuevo la IP al servidor DHCP.
- **PowerShell**: `Get-NetIPConfiguration` muestra un resumen claro de cada adaptador. `Test-NetConnection` comprueba la conectividad con un equipo, e incluso con un puerto concreto (`Test-NetConnection google.com -Port 443`).

Para fijar una IP estática desde PowerShell, primero hay que saber el nombre del adaptador (`Get-NetAdapter`, normalmente `Ethernet`) y después:

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.56.20 -PrefixLength 24
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 8.8.8.8
```

(Sustituye la IP por una del rango de tu red. Si tu red tiene puerta de enlace, se añade con `-DefaultGateway` en la primera orden.)

### El cortafuegos: cuando la red "funciona a medias"

En UD03 hiciste `ping` desde Windows a tu Ubuntu, y funcionó. Si ahora lo haces al revés, **desde Ubuntu a Windows, lo normal es que no responda**, aunque la red esté perfectamente configurada. La causa es el **Firewall de Windows Defender**, que por defecto bloquea las peticiones de `ping` entrantes (y que es más estricto todavía cuando la red está marcada como **Pública** en lugar de **Privada**).

Es una lección de diagnóstico importante: **que un equipo no responda al `ping` no significa que no esté en la red**. Antes de tocar la configuración IP, comprueba si hay un cortafuegos en medio. Para permitir el `ping`, abre **Firewall de Windows Defender con seguridad avanzada** (`wf.msc`) → **Reglas de entrada**, busca **Compartir archivos e impresoras (petición de eco: ICMPv4 de entrada)** y elige **Habilitar regla**. El estudio del cortafuegos lo harás a fondo en el módulo de Seguridad Informática.

> **Vamos a practicar: IP estática y diagnóstico entre tus dos VMs**
>
> **Preparación:** las dos VMs (Ubuntu y Windows) deben estar en el mismo modo de red que configuraste en UD03 para que se vean entre sí.
>
> **Paso a paso:**
>
> 1. En Windows, `ipconfig /all`: anota la IP actual, la máscara, la puerta de enlace, los DNS y si DHCP está habilitado.
> 2. **Configuración → Red e Internet → Ethernet → Asignación de IP → Editar → Manual**, activa IPv4 y fija una IP del mismo rango que la de tu Ubuntu, con su prefijo y su puerta de enlace. Guarda.
> 3. `Get-NetIPConfiguration`: comprueba que la IP nueva se ha aplicado.
> 4. Desde Windows, `ping <IP de tu Ubuntu>`: debe responder.
> 5. Desde Ubuntu, `ping <IP de tu Windows>`: lo más probable es que **no** responda. Anota el resultado.
> 6. En Windows, habilita la regla del cortafuegos **Compartir archivos e impresoras (petición de eco: ICMPv4 de entrada)** desde `wf.msc`. Repite el `ping` desde Ubuntu.
> 7. Vuelve a dejar la red en **Automático (DHCP)** desde Configuración y comprueba con `ipconfig` que la IP vuelve a ser la de antes.
>
> **Reflexiona:** en el paso 5, la red estaba bien configurada en los dos equipos y aun así el `ping` fallaba en un sentido y no en el otro. Si un compañero te dice "no hay red entre las dos máquinas", ¿qué comprobaciones harías, y en qué orden, antes de concluir que la configuración IP está mal?

### Dispositivos y controladores

El **Administrador de dispositivos** (`devmgmt.msc`) es la herramienta para ver todo el hardware que Windows ha detectado y el **controlador** (*driver*) que usa cada dispositivo. Ya sabes desde UD01 que el controlador es el software que permite al sistema operativo hablar con un componente concreto.

Lo primero que hace un técnico al abrirlo es buscar **símbolos de aviso**:

- **Triángulo amarillo con exclamación**: el dispositivo tiene un problema, normalmente un controlador que falta o que falla. En las propiedades del dispositivo, pestaña **General**, aparece un **código de error** que se puede buscar.
- **Flecha hacia abajo**: el dispositivo está **deshabilitado** a propósito.
- **Otros dispositivos → Dispositivo desconocido**: Windows no sabe ni qué es; falta su controlador.

En las propiedades de cada dispositivo, la pestaña **Controlador** permite **Actualizar el controlador**, **Revertir al controlador anterior** (muy útil cuando un dispositivo dejó de funcionar justo después de una actualización), **Deshabilitar** y **Desinstalar**. En una VM verás los controladores de VirtualBox que instalaste con las *Guest Additions* en UD02: el adaptador gráfico, el ratón integrado, etc.

### Impresoras

El asistente de **Configuración → Bluetooth y dispositivos → Impresoras y escáneres → Agregar dispositivo** busca impresoras en la red y conectadas por USB. Si la que quieres no aparece, el enlace **Agregar manualmente** permite indicar una impresora por su dirección IP o por su nombre en la red, igual que el asistente de impresoras que viste en Ubuntu.

Windows incluye además una impresora **virtual**, **Microsoft Print to PDF**, que en lugar de imprimir en papel genera un archivo PDF. Es la forma más sencilla de comprobar que la parte de software de la impresión funciona sin necesidad de una impresora real.

> **Vamos a practicar: dispositivos e impresoras**
>
> **Paso a paso:**
>
> 1. Abre `devmgmt.msc`. Despliega **Adaptadores de pantalla** y **Adaptadores de red**. Anota el nombre de cada dispositivo y, en sus propiedades, pestaña **Controlador**, el fabricante y la fecha del controlador.
> 2. Busca si hay algún dispositivo con triángulo amarillo. En el menú **Ver**, activa **Mostrar dispositivos ocultos** y fíjate en los que aparecen semitransparentes: son dispositivos que estuvieron conectados alguna vez y ya no lo están.
> 3. **Deshabilita** el adaptador de red (clic derecho → **Deshabilitar dispositivo**). Comprueba con `ipconfig` y con un `ping` a internet qué ha pasado. Vuelve a **habilitarlo**.
> 4. En **Impresoras y escáneres**, inicia **Agregar dispositivo** y después **Agregar manualmente** → **Agregar una impresora por medio de una dirección TCP/IP o un nombre de host**. Recorre el asistente con una IP inventada hasta que se detenga, y cancela.
> 5. Abre el Bloc de notas, escribe tu nombre e imprime con **Microsoft Print to PDF**. Guarda el PDF en el Escritorio y ábrelo.
>
> **Comparación opcional en el equipo del aula (solo consulta, sin cambiar nada):** abre también `devmgmt.msc` en el equipo anfitrión, si tu usuario lo permite. Compara los **Adaptadores de pantalla** y los **Adaptadores de red** con los de la VM. Anota qué diferencia ves entre unos y otros y explica por qué.
>
> **Reflexiona:** un usuario te dice que, tras una actualización, el sonido ha dejado de funcionar. En el Administrador de dispositivos, la tarjeta de sonido no tiene ningún aviso. ¿Qué opción del controlador probarías primero y por qué?

---

## 6. Automatización de tareas

### El Programador de tareas

El equivalente a `cron` y `at` en Windows es el **Programador de tareas** (`taskschd.msc`). La diferencia de partida es que no se basa en líneas de texto con cinco campos, sino en un asistente gráfico en el que cada tarea tiene cuatro partes:

- **Desencadenador**: **cuándo** se ejecuta. Además de fechas y horas (una vez, diariamente, semanalmente, mensualmente), admite situaciones que `cron` no contempla directamente: **al iniciar el sistema**, **al iniciar sesión**, **al bloquear o desbloquear**, **cuando el equipo está inactivo** o **cuando se registra un evento concreto** en el sistema.
- **Acción**: **qué** se ejecuta. Casi siempre, **Iniciar un programa**: un ejecutable, un script, o `powershell.exe` con un script como argumento.
- **Condiciones**: requisitos adicionales, por ejemplo "solo si el equipo está conectado a la corriente" o "solo si hay red".
- **Configuración**: qué hacer si la tarea falla, si se pasa la hora porque el equipo estaba apagado, si dura demasiado...

Fíjate en un detalle que en UD03 te obligó a usar un truco: el desencadenador **Semanalmente** tiene un campo **Repetir cada N semanas**. Lo que en `cron` requería comprobar la semana del año con `date +%V`, aquí es una opción del asistente.

Las tareas que creas aparecen en la **Biblioteca del Programador de tareas**, junto a cientos de tareas que el propio Windows usa para su mantenimiento (desfragmentar, buscar actualizaciones, hacer diagnósticos...). Al seleccionar una, la columna **Resultado de la última ejecución** dice si funcionó: `0x0` significa que terminó correctamente, y cualquier otro código indica un error. El **Historial** detallado de cada tarea viene **deshabilitado** de serie; se activa con **Habilitar historial de todas las tareas** en el panel derecho.

### La misma trampa que en Linux: la carpeta de trabajo

En UD03 viste que un script lanzado por `cron` no se ejecuta en tu carpeta personal, y que por eso hay que usar rutas absolutas. **En Windows pasa exactamente lo mismo**: una tarea programada se ejecuta, salvo que digas lo contrario, en `C:\Windows\System32`. Un script que escribe en `registro.txt` sin ruta acabará intentando escribir allí, donde un usuario normal no tiene permiso. La solución es la misma: **rutas absolutas siempre**, o rellenar el campo **Iniciar en (opcional)** de la acción.

### Ejecutar con privilegios: el equivalente al crontab de root

En Linux, las tareas que necesitaban privilegios (como apagar el equipo) iban al crontab de root. En el Programador de tareas, esto se decide en la pestaña **General** de cada tarea:

- **Ejecutar solo cuando el usuario haya iniciado sesión**: la tarea se ejecuta dentro de tu sesión y puede mostrar ventanas. Si no has iniciado sesión, no se ejecuta.
- **Ejecutar tanto si el usuario inició sesión como si no**: se ejecuta en segundo plano, sin ventanas visibles. Pide tu contraseña al guardarla.
- **Ejecutar con los privilegios más altos**: la tarea se ejecuta como administrador, sin el aviso de UAC. Es el equivalente a ponerla en el crontab de root.
- También se puede elegir como usuario de la tarea la cuenta **SYSTEM**, la cuenta interna del sistema con máximos privilegios.

### Scripts de PowerShell y la directiva de ejecución

Un **script de PowerShell** es un archivo de texto con extensión **`.ps1`** que contiene cmdlets, uno detrás de otro, igual que un script `.sh` contenía órdenes de bash. La primera vez que intentes ejecutar uno, lo más probable es que te encuentres este error:

```
No se puede cargar el archivo C:\Scripts\saludo.ps1 porque la ejecución
de scripts está deshabilitada en este sistema.
```

No es un fallo: es la **directiva de ejecución** (*Execution Policy*), una medida de seguridad que en Windows 11 viene en modo **`Restricted`** (no se ejecuta ningún script). Su objetivo es evitar que un usuario ejecute sin querer un script descargado haciendo doble clic. Se consulta con `Get-ExecutionPolicy` y se cambia con:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Con **`RemoteSigned`** se ejecutan los scripts creados en el propio equipo, pero los **descargados de internet** solo si están firmados digitalmente por su autor. Si te descargas un script en el que confías y no está firmado, tendrás que desbloquearlo antes con `Unblock-File .\script.ps1`. `-Scope CurrentUser` aplica el cambio solo a tu usuario, que es lo más prudente.

En una tarea programada, en lugar de cambiar la directiva para todo el sistema, lo habitual es indicarla en la propia acción:

- **Programa o script:** `powershell.exe`
- **Agregar argumentos:** `-NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\saludo.ps1"`

### Programar desde la terminal: `schtasks`

Igual que `crontab -e` frente a un asistente, el Programador de tareas también se maneja desde la terminal con **`schtasks`**, lo que permite crear la misma tarea en muchos equipos con un único comando:

```
schtasks /create /tn "Saludo" /tr "powershell.exe -NoProfile -ExecutionPolicy Bypass -File C:\Scripts\saludo.ps1" /sc weekly /d LUN,MAR,MIE,JUE,VIE /st 09:00
schtasks /query /tn "Saludo" /v /fo list
schtasks /run /tn "Saludo"
schtasks /delete /tn "Saludo" /f
```

Los parámetros más importantes: `/tn` es el nombre de la tarea; `/tr` lo que se ejecuta; `/sc` la frecuencia (`once`, `daily`, `weekly`, `monthly`, `onstart`, `onlogon`...); `/d` los días; `/st` la hora; y `/ru SYSTEM` la ejecuta con la cuenta del sistema. Con `/sc once` se consigue lo mismo que con `at` en Linux: una tarea que se ejecuta una sola vez.

> ⚠️ Los nombres de los días en `/d` dependen del idioma de Windows: en un Windows en español son `LUN, MAR, MIE, JUE, VIE, SAB, DOM`; en uno en inglés, `MON, TUE, WED...`. Si `schtasks` responde que el valor no es válido, prueba con los del otro idioma.

| Linux | Windows |
|---|---|
| `crontab -e` / `crontab -l` | Programador de tareas (`taskschd.msc`) / `schtasks /create` · `schtasks /query` |
| `at 18:00` | `schtasks /create ... /sc once /st 18:00` |
| `sudo crontab -e` (tareas de root) | **Ejecutar con los privilegios más altos** o `/ru SYSTEM` |
| Script `.sh` con `chmod +x` | Script `.ps1` con la directiva de ejecución adecuada |
| Rutas absolutas porque `cron` no arranca en tu carpeta | Rutas absolutas porque la tarea arranca en `C:\Windows\System32` |
| `>> registro.log 2>&1` para dejar constancia | Columna **Resultado de la última ejecución** e historial de la tarea |

> **Vamos a practicar: tu primer script y tu primera tarea**
>
> **Paso a paso:**
>
> 1. Crea la carpeta `C:\Scripts`.
> 2. Abre el Bloc de notas y escribe esta línea (cambia `tu_usuario` por tu usuario real):
>    ```powershell
>    Add-Content -Path "C:\Users\tu_usuario\registro.txt" -Value (Get-Date)
>    ```
>    Guárdalo como `C:\Scripts\registro.ps1`. En el cuadro de guardar, elige **Tipo: Todos los archivos**, o el Bloc de notas lo guardará como `registro.ps1.txt`.
> 3. En PowerShell, ejecuta `C:\Scripts\registro.ps1`. Lee el error de la directiva de ejecución.
> 4. `Get-ExecutionPolicy`, después `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` y vuelve a ejecutar el script. Comprueba con `Get-Content C:\Users\tu_usuario\registro.txt` que ha escrito la fecha.
> 5. Abre `taskschd.msc` → **Crear tarea básica**. Nombre: `Registro`. Desencadenador: **Diariamente**, a una hora de dentro de 2 minutos. Acción: **Iniciar un programa** → Programa: `powershell.exe` → Argumentos: `-NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\registro.ps1"`.
> 6. Espera a la hora indicada y comprueba que `registro.txt` tiene una línea nueva. En la biblioteca, comprueba que el **Resultado de la última ejecución** de la tarea es `0x0`.
> 7. Selecciona la tarea y pulsa **Ejecutar** en el panel derecho: así puedes probar una tarea sin esperar a su hora. Comprueba que aparece otra línea.
>
> **Reflexiona:** si en el script hubieras escrito solo `registro.txt`, sin ruta, el script funcionaría al ejecutarlo a mano desde tu carpeta personal, pero no desde la tarea. ¿Dónde intentaría escribir la tarea, y por qué fallaría?

> **Vamos a practicar: tres tareas con temporalidades distintas**
>
> Igual que en UD03, tu profesor o profesora te va a facilitar un script de copia de seguridad, `backup_personal.ps1`, y tú vas a programarlo junto a otras dos tareas.
>
> **Paso a paso:**
>
> 1. Copia `backup_personal.ps1` a `C:\Scripts`. Si lo has descargado, desbloquéalo con `Unblock-File C:\Scripts\backup_personal.ps1`. Pruébalo una vez a mano y comprueba que crea un `.zip` en `C:\Users\tu_usuario\backups`.
> 2. Prográmalo en el Programador de tareas (con **Crear tarea**, no **Crear tarea básica**, para tener todas las opciones): **Semanalmente**, los **domingos** a las 3:00, **cada 2 semanas**. En **Configuración**, marca **Ejecutar la tarea lo antes posible después de perder un inicio programado** (si a esa hora la VM está apagada, se hará al encenderla).
> 3. Crea un script `C:\Scripts\saludo.ps1` que añada a `C:\Users\tu_usuario\saludo.txt` una línea como "¡Hola tu_usuario! Son las 09:00 del lunes". (Pista: `Get-Date -Format "HH:mm"` da la hora y `(Get-Date).ToString("dddd")` el día de la semana.) Al guardarlo en el Bloc de notas, elige la codificación **UTF-8 con BOM**: Windows PowerShell 5.1 interpreta mal las tildes y la `¡` de un script guardado en UTF-8 normal.
> 4. Prográmalo **desde la terminal** con `schtasks` para que se ejecute **solo los días laborables** a la hora que elijas.
> 5. Crea, en una terminal **como administrador**, una tarea de apagado diario a las 14:55 que se ejecute con la cuenta del sistema: `schtasks /create /tn "Apagado aula" /tr "shutdown /s /t 60" /sc daily /st 14:55 /ru SYSTEM`. Para probarla sin esperar, ejecútala con `schtasks /run /tn "Apagado aula"`; tendrás 60 segundos antes del apagado (y `shutdown /a` lo cancela, si lo necesitas).
> 6. Documenta, con capturas, las tres tareas en la biblioteca del Programador de tareas y la salida de `schtasks /query /tn "Saludo" /v /fo list`.
>
> **Reflexiona:** la tarea de apagado se ejecuta como SYSTEM. ¿Qué pasaría si la hubieras creado como tu usuario, sin privilegios de administrador y con la opción **Ejecutar solo cuando el usuario haya iniciado sesión**, y a las 14:55 nadie hubiera iniciado sesión en el equipo?

---

## Caso práctico integrador

Trabajas en el servicio técnico de una academia. La responsable te comenta lo siguiente sobre el equipo de recepción, con Windows 11 Pro:

- "Desde la actualización de la semana pasada, la impresora no imprime. La he apagado y encendido el ordenador varias veces y sigue igual."
- "Hemos comprado un disco interno para guardar los documentos de los alumnos, pero no aparece en **Este equipo**."
- "Quiero que se haga una copia de los documentos cada viernes a las 20:00, aunque no haya nadie con la sesión iniciada."
- "Además, quiero que tenga los mismos programas que el equipo del despacho."

Con lo aprendido en esta unidad:

1. La responsable ha "apagado y encendido el ordenador varias veces". ¿Por qué eso no equivale a reiniciar en este equipo, y qué deberías comprobar o hacer primero?
2. Si el problema de la impresora persiste tras un reinicio completo, ¿qué herramientas tendrías para volver a la situación anterior a la actualización? Ordénalas de la menos a la más drástica, y explica qué comprobarías en el Administrador de dispositivos.
3. El disco nuevo no aparece en el Explorador. ¿Qué herramienta abrirías y qué cuatro pasos tendrías que dar? ¿Qué sistema de archivos elegirías y por qué?
4. Para la copia de los viernes: ¿qué opciones de la pestaña **General** del Programador de tareas tendrías que configurar, y qué precaución debes tener con las rutas dentro del script?
5. ¿Cómo instalarías en el equipo de recepción exactamente los mismos programas que en el del despacho, sin buscarlos uno a uno?

---

## Resumen de la unidad

- El arranque de Windows sigue la misma cadena que el de Linux: UEFI → Administrador de arranque (configurado en el **BCD**) → `winload.efi` → núcleo (`ntoskrnl.exe`) → servicios → inicio de sesión → `explorer.exe`.
- Con el **inicio rápido** activado, **Apagar** no reinicia el núcleo: **Reiniciar** hace un arranque más completo que apagar y encender. `shutdown /s /t 0` o `Mayús` + Apagar hacen un apagado completo.
- Las **opciones avanzadas de arranque** (WinRE) se abren con `Mayús` + Reiniciar, desde **Configuración → Sistema → Recuperación** o con `shutdown /r /o /t 0`. El **modo seguro** arranca con lo mínimo y sirve para diagnosticar.
- **Bloquear**, **cambiar de usuario** y **cerrar sesión** no son lo mismo: al cambiar de usuario, la primera sesión sigue abierta. `quser` muestra las sesiones abiertas.
- En Windows conviven **CMD** (heredado) y **PowerShell** (moderno), dentro de **Terminal Windows**. PowerShell usa **cmdlets** Verbo-Sustantivo y su tubería pasa **objetos**, no texto. Las tareas de administración requieren abrir la terminal **como administrador**.
- **Configuración** y **Panel de control** conviven; conviene conocer los atajos `.cpl` y `.msc`. Con Windows sin activar, la Personalización queda bloqueada.
- **NTFS** para discos internos, **exFAT** para discos externos compartidos entre sistemas, **FAT32** solo por compatibilidad (límite de 4 GB por archivo), **ReFS** para servidores.
- Un disco nuevo se **inicializa** (GPT), se **particiona**, se **formatea** y se le asigna **letra** o se **monta en una carpeta NTFS vacía**, con Administración de discos o `diskpart`. Windows recuerda la letra solo, sin un archivo equivalente a `fstab`.
- La recuperación es una **escalera**: reiniciar → modo seguro → `DISM` y después `sfc` → Restaurar sistema → desinstalar actualizaciones → Restablecer este PC → reinstalación. Los **puntos de restauración** no protegen los archivos personales y requieren activar la protección del sistema.
- **WinRE** permite reparar sin que Windows arranque. En equipos UEFI, el arranque se repara con `bcdboot`, no con el antiguo `bootrec`. Dentro de WinRE, Windows suele no estar en `C:`.
- **Windows Update** distribuye actualizaciones **de calidad** (mensuales y acumulativas) y **de características** (anuales). Cada actualización tiene un código **KB**.
- Software: instaladores **`.exe`**, paquetes **`.msi`** y **Microsoft Store**. **`winget`** es el gestor de paquetes de Windows, con comandos equivalentes a APT, y `winget export`/`import` permite replicar los programas de un equipo en otro.
- La red se configura desde **Configuración**, `ncpa.cpl` o PowerShell (`Get-NetIPConfiguration`, `New-NetIPAddress`). Un `ping` sin respuesta no significa que no haya red: el **cortafuegos** de Windows bloquea el `ping` entrante por defecto.
- El **Administrador de dispositivos** muestra el estado de cada controlador; **Revertir al controlador anterior** deshace una actualización de controlador problemática.
- El **Programador de tareas** (y `schtasks`) es el equivalente a `cron` y `at`, con desencadenadores por tiempo y por situación. Las tareas arrancan en `C:\Windows\System32`, así que los scripts necesitan **rutas absolutas**. La **directiva de ejecución** controla qué scripts `.ps1` se pueden ejecutar.

## Relación con RA y CE

| CE | Contenido | Dónde se trabaja |
|---|---|---|
| a) | Arranque y parada del sistema, inicio rápido, opciones avanzadas, gestión de sesiones | Apartado 1 |
| b) | Interfaces de usuario: GUI, CMD y PowerShell | Apartado 2 |
| c) | Preferencias del entorno personal | Apartado 2 |
| d) | Sistemas de archivos específicos: NTFS, exFAT, FAT32, ReFS; volúmenes y letras | Apartado 3 |
| e) | Recuperación del sistema operativo: puntos de restauración, `sfc`/`DISM`, WinRE, `bcdboot` | Apartado 3 |
| f) | Actualización del sistema operativo: Windows Update | Apartado 4 |
| g) | Instalación/desinstalación de utilidades: `.exe`, `.msi`, Store, `winget` | Apartado 4 |
| h) | Asistentes de configuración: red y dispositivos | Apartado 5 |
| i) | Automatización de tareas: Programador de tareas, `schtasks`, PowerShell | Apartado 6 |
