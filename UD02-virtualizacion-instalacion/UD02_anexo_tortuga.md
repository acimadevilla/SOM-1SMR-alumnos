# UD02 — Anexo: quitar la "tortuga" de VirtualBox

**Módulo:** Sistemas Operativos Monopuesto · 1º SMR

> **Para qué sirve este anexo.** Si tu máquina virtual va muy lenta y en la barra de estado de su ventana (abajo a la derecha) aparece una **tortuga verde**, VirtualBox está funcionando a través del hipervisor de Windows, en un modo mucho más lento. La introducción explica por qué ocurre; después tienes los comandos listos para copiar y pegar, en orden, para diagnosticarlo y corregirlo.

## Introducción: por qué aparece la tortuga

Antes de lanzar ningún comando conviene entender qué está pasando. Así sabrás qué hace cada paso y por qué a veces no basta con desmarcar Hyper-V.

### 1. Lo que aporta el procesador: VT-x / AMD-V

Un sistema operativo invitado espera tener el control total de la máquina: modificar la memoria, atender interrupciones, ejecutar instrucciones privilegiadas. Si cada una de esas instrucciones tuviera que interceptarla y emularla un programa, la máquina virtual iría lentísima.

Por eso Intel (**VT-x**) y AMD (**AMD-V**) añadieron al procesador un modo especial con dos niveles:

- **Modo raíz:** ahí se ejecuta el **hipervisor**, que es quien manda.
- **Modo no raíz:** ahí se ejecuta el sistema invitado, directamente sobre el procesador real y a velocidad casi nativa.

Cuando el invitado intenta algo delicado, el procesador lo detiene automáticamente y devuelve el control al hipervisor. Es lo que se llama una **salida de VM** (*VM exit*). El hipervisor lo resuelve y devuelve el control al invitado.

**La clave es que solo puede haber un dueño del modo raíz.** Quien lo ocupa primero, al arrancar el equipo, es el hipervisor que manda. Es como un coche con un solo volante.

### 2. Cómo usa VirtualBox el procesador cuando está libre

VirtualBox es un hipervisor de **tipo 2**: se instala como una aplicación sobre Windows. Pero instala también un **controlador en el núcleo** de Windows (`VBoxSup.sys`), y ese controlador es el que pide al procesador entrar en modo raíz.

```
   Invitado (Windows 11 / Ubuntu)         ← modo no raíz, a velocidad nativa
 ─────────────────────────────────
   VirtualBox (VBoxSup.sys)               ← modo raíz: VirtualBox controla VT-x
 ─────────────────────────────────
   Windows anfitrión
 ─────────────────────────────────
   Procesador con VT-x / AMD-V
```

Cada salida de VM la atiende VirtualBox directamente: un viaje corto. Esto es lo que aparece como **«Motor de ejecución: VT-x/AMD-V»**, con el chip azul de la «V».

### 3. Qué cambia cuando se carga el hipervisor de Windows

**Hyper-V** es un hipervisor de **tipo 1**. Cuando está activo, el equipo arranca en este orden:

1. Se carga el hipervisor de Microsoft, que se queda con el modo raíz.
2. Encima de él arranca **el propio Windows anfitrión**, convertido en una máquina virtual más. Microsoft la llama *partición raíz*: es privilegiada, pero ya no está en el modo raíz del procesador.

```
   Invitado de VirtualBox
 ─────────────────────────────────
   VirtualBox (pide ayuda por la API WHP)
 ─────────────────────────────────
   Windows anfitrión = "partición raíz"   ← ¡también virtualizado!
 ─────────────────────────────────
   Hipervisor de Windows                  ← dueño del modo raíz
 ─────────────────────────────────
   Procesador con VT-x / AMD-V
```

Ahora, cuando el controlador de VirtualBox intenta entrar en modo raíz, no puede: el volante ya lo tiene otro. VirtualBox tiene dos opciones, no arrancar o pedir ayuda. Desde la versión 6 elige pedir ayuda: usa la **Plataforma del hipervisor de Windows** (**WHP**), una API con la que Microsoft permite a programas externos pedirle al hipervisor de Windows que ejecute sus máquinas virtuales. Es la **«API nativa»** que ves en la información de sesión, y la **tortuga**.

### 4. Por qué va tan lento

En el modo de la tortuga, cada salida de VM da un viaje mucho más largo:

> invitado → hipervisor de Windows → Windows anfitrión → VirtualBox (que decide qué hacer) → Windows anfitrión → hipervisor de Windows → invitado

Un invitado normal provoca **miles de salidas por segundo**, por el disco, la pantalla, la red, los temporizadores o las interrupciones. Si cada una cuesta varias veces más, el resultado es:
- instalaciones eternas;
- pantalla en negro mientras Windows inicializa el vídeo;
- el ratón a saltos.

A esto se suma que la parte de VirtualBox que trabaja sobre WHP está mucho menos optimizada que la que usa VT-x directamente. Lo mismo ocurre con VMware y otros hipervisores de tipo 2, que también caen a WHP cuando el hipervisor de Windows está activo.

### 5. Por qué no basta con desmarcar Hyper-V

Lo que provoca la tortuga no es la característica «Hyper-V», sino que **el hipervisor de Windows esté cargado**. Y a ese hipervisor no lo usa solo Hyper-V:

| Quién lo usa | Para qué |
|---|---|
| Hyper-V | Ejecutar máquinas virtuales de Microsoft |
| Plataforma de máquina virtual | WSL 2 y Docker Desktop (la VM Linux que ejecutan por debajo) |
| Espacio aislado de Windows | Un Windows desechable para probar programas |
| **Seguridad basada en virtualización (VBS)** | Crear una zona de memoria aislada, que ni el propio núcleo de Windows puede tocar, donde guardar lo más sensible |

Esa zona aislada de la VBS es donde funcionan:
- la **integridad de memoria** (HVCI), que comprueba que el código que se carga en el núcleo es legítimo;
- **Credential Guard**, que guarda ahí las contraseñas y los tickets del dominio para que un malware no pueda robarlos de la memoria.

Windows 11 activa la VBS **por defecto**, y Credential Guard también en las ediciones Pro, Education y Enterprise unidas a un dominio. Ninguna de las dos aparece en «Características de Windows». Es lo que se comprobó en un equipo del aula:
- las características estaban desmarcadas;
- la integridad de memoria estaba desactivada;
- `hypervisorlaunchtype` estaba en Off;
- y aun así `VirtualizationBasedSecurityStatus` daba **2**. La VBS seguía obligando a Windows a cargar su hipervisor.

Por eso este anexo no se queda en Hyper-V: también desactiva la VBS y Credential Guard (paso 2).

### 6. En resumen

- El procesador solo admite **un dueño** de VT-x/AMD-V a la vez.
- Si al arrancar se carga el **hipervisor de Windows**, por Hyper-V, WSL 2, el Espacio aislado o la **VBS**, el dueño es él.
- VirtualBox no puede usar VT-x directamente y trabaja **a través de él** con la API WHP: es la **tortuga**. Funciona, pero cada salida de VM tarda mucho más.
- Para quitarla hay que conseguir que **nada cargue ese hipervisor** al arrancar, y no basta con desmarcar Hyper-V: también la VBS y Credential Guard.
- Hay que elegir: **rendimiento de VirtualBox** o **WSL 2, Hyper-V y la protección de la VBS**. No se pueden tener las dos cosas a pleno rendimiento.

> **Relación con otros módulos.**
> - **Seguridad Informática:** qué protege Credential Guard (robo de credenciales con herramientas tipo Mimikatz) y por qué desactivarlo es una decisión de seguridad, no solo técnica.
> - **Montaje y Mantenimiento:** la opción «Intel Virtualization Technology» o «SVM Mode» de la UEFI es la que habilita VT-x o AMD-V. Si está desactivada, no hay volante para nadie.

## Antes de empezar

- **Todo se hace en el equipo anfitrión**, el Windows 11 donde está instalado VirtualBox, no dentro de la máquina virtual.
- **Necesitas una cuenta de administrador.** En los equipos del aula, hazlo solo cuando lo indique tu profesor. En tu ordenador personal, lee antes el apartado «Lo que hay que saber».
- **Cierra VirtualBox** con todas las máquinas virtuales apagadas.
- **Copia los comandos del archivo de texto `UD02_anexo_tortuga_comandos.txt`**, que está junto a este anexo en el repositorio. Ábrelo con el Bloc de notas: tiene todos los comandos en el mismo orden, cada uno en una sola línea. También puedes copiarlos desde la versión web de este anexo (el archivo `.md`), con el botón de copiar de cada bloque. **No los copies del PDF:** las líneas largas se parten y el comando falla.

### Abre PowerShell como administrador

Haz **clic derecho en el botón Inicio** → **Terminal (Administrador)** y acepta el aviso de Control de cuentas de usuario. Se abre **PowerShell**, no CMD: todos los comandos de este anexo están pensados para PowerShell.

Para pegar un comando en la terminal, usa **clic derecho** o **Ctrl + V**, y pulsa **Intro**.

> **Ojo con las llaves.** En PowerShell, las llaves `{ }` tienen un significado especial. Por eso un comando como `bcdedit /enum {current}`, que funciona en CMD, en PowerShell da el error «El tipo de entrada especificado no es válido». En PowerShell hay que escribirlo con comillas: `bcdedit /enum "{current}"`.

## Paso 1. Diagnóstico: ¿qué está cargando el hipervisor?

Ejecuta estos comandos uno a uno y anota lo que sale. Así sabrás qué está pasando antes de cambiar nada.

**1.a. ¿Está en marcha la seguridad basada en virtualización?**

```powershell
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard | Select-Object VirtualizationBasedSecurityStatus, SecurityServicesRunning
```

| Valor | Qué significa |
|---|---|
| `VirtualizationBasedSecurityStatus` = **2** | La seguridad basada en virtualización está **en ejecución**. Es la causa más habitual de la tortuga |
| `VirtualizationBasedSecurityStatus` = **0** | Está desactivada |
| `SecurityServicesRunning` contiene **1** | Está activo **Credential Guard**, que protege las contraseñas guardadas en memoria |
| `SecurityServicesRunning` contiene **2** | Está activa la **integridad de memoria** |

**1.b. ¿Está configurado el arranque del hipervisor?**

```powershell
bcdedit /enum "{current}"
```

Busca la línea `hypervisorlaunchtype`. Si pone **Auto** o no aparece, el hipervisor arranca con Windows. Si pone **Off**, este paso ya está hecho.

**1.c. ¿Hay alguna característica de Windows que lo active?**

```powershell
Get-WindowsOptionalFeature -Online | Where-Object { $_.State -eq "Enabled" -and $_.FeatureName -match "Hyper-V|VirtualMachinePlatform|HypervisorPlatform|DisposableClientVM" } | Select-Object FeatureName
```

Si no sale ninguna línea, no hay ninguna característica activa que cargue el hipervisor.

**1.d. ¿Impone el centro una directiva sobre esto?**

```powershell
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceGuard"
```

- Si responde **«El sistema no ha podido encontrar la clave o el valor del registro especificados»**, no hay directiva. Puedes seguir con el paso 2.
- Si **devuelve valores**, la configuración la impone quien administra el equipo, por ejemplo el centro. **No sigas.** Avisa a tu profesor: si la cambias, la directiva la volverá a imponer, y además no te corresponde a ti cambiarla.

## Paso 2. Desactivar lo que carga el hipervisor

Cada comando debe terminar sin errores en rojo. Los de `reg add` responden «La operación se completó correctamente».

**2.a. Desactivar las características de Windows que cargan el hipervisor:** Hyper-V, Plataforma de máquina virtual, Plataforma del hipervisor de Windows y Espacio aislado de Windows. Si alguna no existe en tu edición de Windows, simplemente se salta.

```powershell
foreach ($f in "Microsoft-Hyper-V-All","VirtualMachinePlatform","HypervisorPlatform","Containers-DisposableClientVM") { Disable-WindowsOptionalFeature -Online -FeatureName $f -NoRestart -ErrorAction SilentlyContinue | Out-Null }
```

Puede tardar un minuto. Cuando vuelva a aparecer el cursor, ha terminado.

**2.b. Impedir que el hipervisor arranque con Windows:**

```powershell
bcdedit /set hypervisorlaunchtype off
```

**2.c. Desactivar la integridad de memoria.** Es lo mismo que desactivarla en Seguridad de Windows → Seguridad del dispositivo → Aislamiento del núcleo.

```powershell
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\HypervisorEnforcedCodeIntegrity" /v Enabled /t REG_DWORD /d 0 /f
```

**2.d. Desactivar la seguridad basada en virtualización:**

```powershell
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v EnableVirtualizationBasedSecurity /t REG_DWORD /d 0 /f
```

**2.e. Desactivar Credential Guard.** Windows 11 lo activa por defecto en las ediciones Pro, Education y Enterprise cuando el equipo pertenece a un dominio, y no aparece en ningún menú. Son dos comandos:

```powershell
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v LsaCfgFlags /t REG_DWORD /d 0 /f
```

```powershell
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceGuard" /v LsaCfgFlags /t REG_DWORD /d 0 /f
```

## Paso 3. Reiniciar

Ningún cambio surte efecto hasta reiniciar. Usa **Inicio → Iniciar/Apagar → Reiniciar**, no «Apagar» seguido de encender. Con el inicio rápido de Windows, apagar no recarga del todo el arranque.

## Paso 4. Comprobar el resultado

Abre de nuevo **Terminal (Administrador)** y repite el diagnóstico:

```powershell
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard | Select-Object VirtualizationBasedSecurityStatus, SecurityServicesRunning
```

Debe salir **`VirtualizationBasedSecurityStatus` = 0**.

Después abre VirtualBox y arranca la máquina virtual:
- En la barra de estado, la tortuga ha cambiado por un **chip azul con una «V»**.
- En **Máquina → Información de sesión**, la línea **«Motor de ejecución»** dice **«VT-x/AMD-V»**.

> **No te confundas:** la línea «Interfaz de paravirtualización: Hyper-V» es normal en una VM con Windows y no indica ningún problema. La que importa es la del motor de ejecución.

## Si la tortuga sigue ahí

| Lo que ves | Causa probable | Qué hacer |
|---|---|---|
| Tras reiniciar, `hypervisorlaunchtype` vuelve a **Auto** o las características vuelven a estar activadas | El equipo **restaura su configuración** en cada arranque, o una directiva la impone | No es algo que puedas arreglar tú: avisa a tu profesor |
| `VirtualizationBasedSecurityStatus` sigue en **2** y `SecurityServicesRunning` contiene **1** | **Credential Guard bloqueado en el firmware UEFI**: el registro no basta para desactivarlo | Hace falta una herramienta de Microsoft que pide confirmación al arrancar. Avisa a tu profesor |
| El paso 1.d devuelve valores | Una **directiva** del centro o de la empresa impone la configuración | No la cambies. Lo resuelve quien administra el equipo |
| Ya no hay tortuga, pero la VM sigue lenta | Pocos recursos asignados a la VM | Asigna al menos **4 GB de RAM** y **2 procesadores** a la VM de Windows 11 |
| Ya no hay tortuga, pero la pantalla se queda en negro tras el primer reinicio de la instalación | Configuración de pantalla de la VM | Con la VM apagada, en **Configuración → Pantalla**: controlador **VBoxSVGA**, **128 MB** de vídeo y aceleración 3D **desactivada** |

Aunque no consigas quitar la tortuga, **puedes seguir trabajando**: VirtualBox funciona igual, solo que más despacio. Dale a la VM algo más de RAM y de procesadores y ten paciencia durante la instalación.

## Lo que hay que saber

- **Rebajas la seguridad del equipo.** La integridad de memoria y Credential Guard protegen Windows frente a ciertos programas maliciosos y frente al robo de contraseñas. En tu ordenador personal, desactivarlas es una decisión razonable si necesitas VirtualBox a buen rendimiento, pero tienes que saber que lo haces. En un equipo del centro o de una empresa, **no se toca sin permiso** del responsable informático: es una norma básica de cualquier técnico.
- **Efectos secundarios:** mientras el hipervisor esté desactivado, **no funcionarán WSL 2, el Espacio aislado de Windows ni Hyper-V**. Docker Desktop para Windows tampoco, porque usa WSL 2.
- **Es reversible.** Con los comandos del apartado siguiente, el equipo vuelve a quedar como estaba.

## Volver a dejarlo como estaba

En **Terminal (Administrador)**, ejecuta estos comandos uno a uno y reinicia. Si alguno dice que no encuentra el valor, no pasa nada: es que ese valor no existía.

```powershell
bcdedit /set hypervisorlaunchtype auto
```

```powershell
reg delete "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v EnableVirtualizationBasedSecurity /f
```

```powershell
reg delete "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v LsaCfgFlags /f
```

```powershell
reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceGuard" /v LsaCfgFlags /f
```

Para volver a activar la integridad de memoria, ve a **Seguridad de Windows → Seguridad del dispositivo → Aislamiento del núcleo** y actívala desde allí. Las características de Windows que necesites, como la Plataforma de máquina virtual para WSL 2, se vuelven a marcar en `optionalfeatures`.
