# UD02 — Anexo: quitar la "tortuga" de VirtualBox

**Módulo:** Sistemas Operativos Monopuesto · 1º SMR

> **Para qué sirve este anexo.** Si tu máquina virtual va muy lenta y en la barra de estado de su ventana (abajo a la derecha) aparece una **tortuga verde**, VirtualBox está funcionando a través del hipervisor de Windows, en un modo mucho más lento. La explicación de por qué ocurre está en los apuntes, epígrafe 2, apartado «VirtualBox y el hipervisor de Windows: la tortuga». Aquí tienes los comandos listos para copiar y pegar, en orden, para diagnosticarlo y corregirlo.

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
