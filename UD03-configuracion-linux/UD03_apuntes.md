# UD03 — Configuración de Linux

**Módulo:** Sistemas Operativos Monopuesto · 1º SMR

## Introducción

En UD02 instalaste tu máquina virtual con Ubuntu, la actualizaste y aprendiste a protegerla con instantáneas antes de hacer cambios arriesgados. Esta unidad no vuelve a instalar nada: te enseña a **vivir con ese sistema el día a día**, que es lo que de verdad va a ocupar la mayor parte del trabajo de un técnico de sistemas — arrancarlo, trabajar en el entorno que prefieras, mantenerlo actualizado, recuperarlo si algo falla y automatizar lo que se repite.

Al terminar esta unidad debes ser capaz de:

- Explicar el proceso de arranque de Linux (GRUB, systemd) y gestionar sesiones de usuario.
- Diferenciar la línea de comandos del entorno gráfico, e instalar, cambiar y personalizar varios entornos de escritorio en el mismo sistema.
- Gestionar sistemas de archivos específicos de Linux: montaje manual y automático mediante `/etc/fstab`.
- Aplicar métodos de recuperación del sistema operativo ante fallos de arranque, incluyendo la reparación de GRUB.
- Mantener el sistema actualizado y gestionar la instalación y desinstalación de software con APT, con nociones comparadas de DNF.
- Usar asistentes de configuración de red y dispositivos.
- Automatizar tareas del sistema con `cron` y `at`.

---

## 1. Arranque y sesiones

### Qué pasa entre que arranca GRUB y ves tu escritorio

En UD02 viste que, tras el firmware (BIOS/UEFI), el gestor de arranque (GRUB en Linux) carga el sistema operativo elegido. Pero "cargar el sistema operativo" no es un único paso: GRUB carga el **núcleo** (*kernel*) de Linux, y el núcleo, a su vez, arranca el primer proceso que se ejecuta en espacio de usuario: **systemd**, con el identificador de proceso (PID) número 1.

systemd es el **sistema de inicio** (*init system*) que usan hoy la inmensa mayoría de distribuciones Linux, incluida Ubuntu. Su trabajo es arrancar el resto de servicios y procesos del sistema, respetando el orden y las dependencias entre ellos: la red no puede arrancar antes de que existan las interfaces de red, un servicio que depende de una base de datos no puede arrancar antes que esa base de datos, y así sucesivamente.

### Unidades y targets

systemd organiza todo lo que gestiona en **unidades** (*units*). Una unidad `.service` es un servicio; una unidad `.mount` es un punto de montaje; una unidad `.timer` es una tarea programada (la verás en el epígrafe 6). Un **target** (`.target`) es un conjunto de unidades que definen un "estado" concreto del sistema — el equivalente moderno a los antiguos *runlevels* que usaban los sistemas de inicio anteriores a systemd.

Los dos targets que más te interesan aquí son:

- **`multi-user.target`**: el sistema arranca completo, con red y todos los servicios, pero **sin interfaz gráfica** — solo modo texto.
- **`graphical.target`**: incluye todo lo anterior, más el gestor de pantalla y el entorno de escritorio.

El comando `systemctl` es la herramienta central para consultar y gestionar unidades y targets. `systemctl get-default` muestra el target por defecto; `systemctl set-default multi-user.target` lo cambia de forma permanente; `systemctl isolate multi-user.target` cambia el estado **ahora mismo**, sin tocar la configuración de arranque — es completamente reversible con `systemctl isolate graphical.target`, sin necesidad de reiniciar.

### `journalctl`: ver qué ha pasado en el arranque

systemd registra toda la actividad del sistema en un registro centralizado llamado el **journal**. El comando `journalctl -b` muestra los mensajes del arranque actual; `journalctl -b -1`, los del arranque anterior. Es tu primera herramienta de diagnóstico real del curso, y la vas a volver a usar en unidades posteriores para investigar problemas de servicios.

### Sesiones de usuario

Una **sesión** es el periodo en el que un usuario autenticado tiene procesos asociados a su cuenta, ya sea en una terminal (TTY) o de forma gráfica. Puede haber **varias sesiones activas a la vez**: por ejemplo, tú conectado por pantalla gráfica y, al mismo tiempo, alguien conectado por SSH desde otro equipo, o dos usuarios distintos en TTYs diferentes. `loginctl list-sessions` muestra todas las sesiones activas del sistema; `loginctl show-session <ID>` da el detalle de una en concreto.

### Cerrar sesión, bloquear, apagar y reiniciar no son lo mismo

Es un error muy habitual mezclar estos cuatro conceptos:

- **Cerrar sesión** (*logout*): vuelve a la pantalla de inicio de sesión. El sistema sigue completamente encendido.
- **Bloquear**: la sesión sigue activa exactamente donde la dejaste, solo que pide contraseña para volver a verla.
- **Apagar** (`systemctl poweroff` o `shutdown -h now`): el equipo se apaga por completo.
- **Reiniciar** (`systemctl reboot`): el equipo se apaga y vuelve a arrancar.

### Para practicar

**Actividad:** sobre tu VM Ubuntu, consulta el target de arranque por defecto, cámbialo temporalmente a modo texto y vuelve al modo gráfico sin reiniciar. Después, consulta con `journalctl -b` los mensajes del arranque actual y localiza la línea en la que se menciona el sistema de archivos raíz montándose.

**Ejemplo resuelto:** `systemctl get-default` → `graphical.target`. `systemctl isolate multi-user.target` → la pantalla pasa a modo texto. `systemctl isolate graphical.target` → vuelve el escritorio, sin haber reiniciado en ningún momento.

> **Vamos a practicar: sesiones activas en tu propio sistema**
>
> Ejecuta `loginctl list-sessions` en tu VM Ubuntu y localiza cuál es tu sesión actual. Después, responde por escrito: ¿qué diferencia real hay, a nivel de systemd, entre `multi-user.target` y `graphical.target`? No te quedes en "uno tiene interfaz gráfica y el otro no": explica la relación de dependencia entre ambos.

---

## 2. Interfaces de usuario y entornos de escritorio

### Línea de comandos y entorno gráfico: cuándo conviene cada una

Ya viste en UD01 que el shell puede ser en texto (línea de comandos) o en gráficos. Ninguna sustituye por completo a la otra: la **línea de comandos** (CLI, *Command Line Interface*) es más rápida para tareas repetitivas o automatizables, funciona igual en local que conectado por SSH a un equipo remoto, y consume muy pocos recursos. El **entorno gráfico** (GUI, *Graphical User Interface*) tiene una curva de aprendizaje menor y resulta más intuitivo para tareas exploratorias. Un técnico competente se mueve entre ambas según lo que la tarea concreta le pida, no por preferencia fija.

### Qué es exactamente un entorno de escritorio

Un **entorno de escritorio** (*Desktop Environment*, DE) es el conjunto de componentes gráficos que conforman la experiencia visual e interactiva de un usuario: el gestor de ventanas, el panel o barra de tareas, el gestor de archivos gráfico, las notificaciones, el tema visual. No es lo mismo que "el shell gráfico" en abstracto que viste en UD01 — aquí hablamos de implementaciones concretas, y distintas entre sí, que puedes instalar y comparar.

Los más habituales que te vas a encontrar en el sector son:

- **GNOME**: el que trae Ubuntu por defecto. Basado en GTK, con un diseño minimalista pensado para pantalla completa y gestos.
- **KDE Plasma**: basado en Qt, muy personalizable, con un aspecto más tradicional de escritorio y barra de tareas.
- **XFCE**: ligero, pensado para hardware limitado, visualmente sencillo.
- **MATE**: continuación del clásico GNOME 2, con un escritorio de estilo tradicional.

Ninguno es "mejor" en abstracto — la elección depende de gustos, del hardware disponible y del caso de uso, exactamente igual que viste con los tipos de núcleo del sistema operativo en UD01.

### El gestor de pantalla y la elección de sesión

El **gestor de pantalla** (*display manager*) es el que muestra la pantalla de inicio de sesión gráfica y arranca el entorno de escritorio correspondiente. GDM está asociado tradicionalmente a GNOME, SDDM a KDE Plasma, y LightDM es una opción ligera pensada para funcionar bien con cualquier DE.

Aquí está la parte que más sorprende la primera vez: **puedes tener varios entornos de escritorio instalados a la vez en el mismo sistema**, sin que uno sustituya al otro. Cuando hay más de uno instalado, la propia pantalla de inicio de sesión ofrece un icono o un menú para elegir con cuál quieres entrar esa vez — sin necesidad de desinstalar nada.

Instalar un segundo entorno se hace igual que instalar cualquier otro software, con el gestor de paquetes que verás formalizado en el epígrafe 4: por ejemplo, `sudo apt install xubuntu-desktop` instala XFCE completo, con su conjunto de aplicaciones por defecto (el paquete `xfce4`, más minimalista, instala solo el núcleo del entorno sin esas aplicaciones añadidas).

### Para practicar

**Actividad:** instala un segundo entorno de escritorio en tu VM Ubuntu (XFCE, KDE Plasma o MATE, según se te indique), usando el meta-paquete completo. Cierra sesión y, desde la pantalla de inicio, entra con el nuevo entorno.

**Ejemplo resuelto:** tras `sudo apt install xubuntu-desktop` y reiniciar la sesión, en la pantalla de login aparece un icono de engranaje junto al campo de contraseña; al pulsarlo, se despliega la lista de sesiones disponibles (GNOME y XFCE), y eliges la que quieras usar en ese inicio de sesión concreto.

> **Vamos a practicar: personaliza y compara**
>
> Dentro de cada uno de tus dos entornos de escritorio, localiza dónde se cambian el fondo de pantalla, el tema y los iconos (por ejemplo, "Ajustes" en GNOME frente a "Configuración del sistema" en KDE Plasma), y personaliza al menos dos aspectos en cada uno. Después, compara cuánta memoria RAM consume cada entorno recién iniciado (puedes usar el monitor de recursos gráfico) y completa una tabla con: RAM en reposo, primera impresión visual, y dónde está la configuración de personalización en cada uno.
>
> **Reflexiona:** si desinstalaras ahora mismo el entorno que no estás usando, ¿qué riesgo real correrías? ¿Por qué es más seguro simplemente no usarlo que desinstalarlo?

---

## 3. Sistemas de archivos y recuperación

### Montar un sistema de archivos, paso a paso

"Montar" un dispositivo o partición significa asociarlo a un punto concreto del árbol de directorios, para poder acceder a su contenido a través de esa ruta. `mount` monta manualmente (por ejemplo, `sudo mount /dev/sdb1 /mnt`); `umount` desmonta. Los puntos de montaje típicos para dispositivos externos son `/mnt` y `/media`.

### `/etc/fstab`: qué se monta solo en cada arranque

Montar a mano cada vez que arrancas sería muy poco práctico para los discos que usas siempre. El archivo **`/etc/fstab`** resuelve esto: define qué se monta automáticamente en cada arranque, sin intervención manual. Cada línea sigue esta sintaxis:

```
<dispositivo>  <punto de montaje>  <tipo de sistema de archivos>  <opciones>  <dump>  <pass>
```

Se recomienda identificar el dispositivo por su **UUID** (un identificador único y estable para esa partición concreta) en vez de por su nombre (`/dev/sdaX`). La razón es puramente práctica: el nombre de dispositivo puede cambiar entre arranques si se añaden o quitan discos, mientras que el UUID no cambia nunca. El comando `blkid` muestra el UUID de cada partición del sistema.

### Cuando el arranque falla: no es el fin del mundo

Un error de sintaxis en `/etc/fstab` puede hacer que el sistema no complete su arranque con normalidad, y entre en un **modo de emergencia** (`emergency.target` de systemd) o en el modo de recuperación del menú avanzado de GRUB, que ya viste por encima en UD02. Es importante que interiorices esto: **no se ha perdido ningún dato**, el sistema simplemente no puede continuar hasta que se corrija la configuración que le impide montar correctamente todo lo que tiene indicado.

El procedimiento habitual es: leer el mensaje de error que muestra la shell de emergencia (suele indicar exactamente qué línea o qué dispositivo ha fallado), acceder con la contraseña de administración, editar `/etc/fstab` con un editor de texto en terminal (`nano` o `vi`) para corregir el error, y reiniciar de nuevo para comprobar.

**Aviso importante, relacionado con lo visto en UD01 sobre journaling:** el journaling de ext4 protege la consistencia del sistema de archivos ante cortes de luz o interrupciones bruscas — pero **no protege de un error humano de configuración** como un `fstab` mal escrito. Son dos problemas de naturaleza completamente distinta, y confundirlos es un error conceptual habitual.

### Reparar GRUB cuando el propio gestor de arranque falla

Si el problema no es `fstab` sino el propio gestor de arranque (una situación que ya viste en UD02 con el dual boot), el procedimiento estándar es más laborioso: arrancar desde un live USB, montar la partición raíz del sistema instalado, montar también dentro `/dev`, `/proc` y `/sys`, hacer `chroot` a ese sistema montado (una forma de "entrar" en él como si fuera el sistema en marcha) y ejecutar `grub-install` y `update-grub` desde dentro. Tu profesor o profesora te lo mostrará en directo, ya que es un procedimiento con muchos pasos.

### Para practicar

**Actividad:** añade un disco o partición adicional a `/etc/fstab`, identificándolo por su UUID, y comprueba que se monta sin reiniciar con `sudo mount -a`.

**Ejemplo resuelto:** `blkid` muestra `/dev/sdb1: UUID="1a2b3c4d-..." TYPE="ext4"`. Se añade a `/etc/fstab` la línea `UUID=1a2b3c4d-... /mnt/datos ext4 defaults 0 2`. Tras `sudo mkdir -p /mnt/datos && sudo mount -a`, el disco aparece montado en `/mnt/datos` sin necesidad de reiniciar.

> **Vamos a practicar: rompe y repara `fstab`, de forma controlada**
>
> Antes de nada, toma una instantánea de tu VM con el nombre "Antes de romper fstab" — es exactamente el tipo de cambio arriesgado para el que aprendiste a usar snapshots en UD02.
>
> Provoca deliberadamente un error de sintaxis en `/etc/fstab` (por ejemplo, cambia un UUID por uno inventado) y reinicia la VM. Documenta con capturas el mensaje de error mostrado, el proceso completo de diagnóstico y corrección, y la comprobación final de que el sistema vuelve a arrancar con normalidad.
>
> **Reflexiona:** explica en 3-4 líneas por qué esta incidencia no tiene nada que ver con el journaling de ext4 que estudiaste en UD01.

---

## 4. Gestión de software: actualización e instalación

### `apt update` y `apt upgrade` no son lo mismo

Es, con diferencia, la confusión más habitual de este bloque, así que conviene fijarla bien desde el principio: **`apt update` no instala nada**. Su única función es refrescar el índice local de paquetes disponibles, consultando los repositorios configurados — es como actualizar el catálogo de una tienda para saber qué hay disponible. **`apt upgrade`** es el que de verdad instala las actualizaciones de los paquetes ya instalados en tu sistema, usando ese índice ya refrescado. Por eso casi siempre se ejecutan juntos y en ese orden: `sudo apt update && sudo apt upgrade`.

`apt full-upgrade` (también llamado `dist-upgrade`) hace lo mismo que `upgrade`, pero permitiendo además instalar o eliminar paquetes si hace falta para resolver dependencias mayores entre versiones.

### De dónde vienen los paquetes: repositorios y PPA

APT consulta una lista de **repositorios** configurada en `/etc/apt/sources.list` (y en los archivos dentro de `/etc/apt/sources.list.d/`) para saber dónde buscar el software disponible. Un **PPA** (*Personal Package Archive*) es un repositorio adicional, normalmente mantenido por terceros, que añade software que no está en los repositorios oficiales de Ubuntu.

Añadir un PPA equivale a confiar en quien lo mantiene — exactamente el mismo criterio de seguridad que ya viste con el origen de las ISOs en UD02: solo se deben añadir repositorios de fuentes conocidas y fiables, nunca por pura comodidad de "que funcione ya".

### Instalar y desinstalar: la diferencia entre `remove` y `purge`

- `apt install <paquete>` instala el paquete indicado.
- `apt remove <paquete>` lo desinstala, pero **conserva sus archivos de configuración**, por si decides reinstalarlo más adelante.
- `apt purge <paquete>` lo desinstala y **también elimina esos archivos de configuración**.
- `apt autoremove` limpia paquetes que quedaron instalados solo como dependencia de otro, y que ya no necesita ningún paquete instalado.

### Instalar un `.deb` suelto

Cuando descargas un archivo `.deb` manualmente, sin pasar por un repositorio, se instala con `sudo dpkg -i paquete.deb`. Si ese paquete necesita otros paquetes que no tienes instalados, `dpkg` falla y deja el sistema con "dependencias rotas" — se soluciona con `sudo apt --fix-broken install`, que hace que APT busque y resuelva automáticamente esas dependencias en los repositorios configurados.

### DNF y `.rpm`: el mismo problema, otra familia de distribuciones

En la familia de distribuciones Fedora/RHEL, el gestor de paquetes equivalente a APT es **DNF**, que trabaja con paquetes `.rpm` en vez de `.deb`. No vas a practicar con DNF en el aula porque no tienes una VM de esa familia instalada, pero conviene que conozcas la equivalencia, para no quedarte bloqueado si en el futuro te encuentras con una distribución de este tipo:

| APT (Debian/Ubuntu) | DNF (Fedora/RHEL) |
|---|---|
| `apt update` | `dnf check-update` |
| `apt upgrade` | `dnf update` |
| `apt install` | `dnf install` |
| `dpkg -i paquete.deb` | `rpm -i paquete.rpm` |

### Para practicar

**Actividad:** instala una utilidad nueva desde el repositorio oficial (por ejemplo, `htop`), compruébala en funcionamiento y desinstálala con `purge`.

**Ejemplo resuelto:** `sudo apt install htop` la instala; ejecutando `htop` se comprueba que funciona; `sudo apt purge htop` la desinstala eliminando también su configuración — si existiera algún archivo de configuración personalizado en `~/.config/htop/`, tendrías que eliminarlo aparte, porque `purge` solo limpia la configuración a nivel de sistema del paquete, no la de cada usuario.

> **Vamos a practicar: un `.deb` suelto y sus dependencias**
>
> Descarga un archivo `.deb` de una fuente oficial fiable e instálalo con `dpkg -i`. Si aparecen dependencias rotas (es un resultado esperado, no un fallo tuyo), documenta cómo las resuelves con `apt --fix-broken install`.
>
> **Reflexiona:** ¿por qué crees que `dpkg -i` por sí solo no resuelve dependencias, mientras que `apt install` sí lo hace automáticamente?

---

## 5. Asistentes de configuración: red y dispositivos

### Qué es un asistente de configuración, y por qué existe

Editar archivos de configuración a mano —como acabas de hacer con `/etc/fstab`— es potente, pero propenso a errores. Un **asistente de configuración** es una herramienta, gráfica o de texto interactivo, que guía paso a paso una configuración compleja ofreciendo solo opciones válidas y comprobando automáticamente lo que introduces, reduciendo así el margen de error a cambio de algo menos de control fino.

### Red: NetworkManager, `nmcli` y `nmtui`

**NetworkManager** es el servicio que gestiona de forma centralizada las conexiones de red en Ubuntu. Puedes acceder a él de tres formas: desde el applet gráfico del escritorio, desde `nmtui` (una interfaz de texto interactiva, tipo menú, cómoda para una configuración puntual) o desde `nmcli` (línea de comandos pura, pensada para usarse dentro de scripts y tareas automatizadas — la volverás a ver relacionada con el epígrafe 6).

Por debajo de todo esto, Ubuntu usa **Netplan** como capa de configuración declarativa (archivos `.yaml` en `/etc/netplan/`) que NetworkManager o systemd-networkd se encargan de aplicar — no necesitas tocarlo directamente para el trabajo habitual, pero conviene que sepas que existe.

### Dispositivos: impresoras y Bluetooth

El gestor de impresoras de Ubuntu, basado en **CUPS** (*Common UNIX Printing System*), permite añadir una impresora local o de red mediante un asistente gráfico, sin tener que configurar CUPS a mano. El applet de Bluetooth funciona de forma parecida para emparejar dispositivos.

### Para practicar

**Actividad:** consulta las conexiones de red activas de tu VM con `nmcli connection show`, y cambia la configuración de una interfaz de DHCP a una IP estática usando `nmtui`.

**Ejemplo resuelto:** `nmcli connection show` lista las conexiones configuradas y su estado. Desde `nmtui` → "Editar una conexión", se cambia el método IPv4 de "Automático" a "Manual" y se introduce una dirección dentro del rango que permite el modo de red de la VM (visto en UD02). Tras guardar y reactivar la conexión, `ping` a otra VM o al anfitrión confirma que funciona.

> **Vamos a practicar: red y modos de VirtualBox, todo junto**
>
> Vuelve a poner tu VM en DHCP. Después, responde: ¿qué modo de red de VirtualBox (de los que viste en UD02: NAT, Red NAT, Adaptador puente, Solo anfitrión, Red interna) es imprescindible para que una IP estática configurada dentro de la VM tenga sentido, y por qué? No te quedes en "hace falta tener red": explica la relación concreta entre el modo elegido en VirtualBox y la configuración que acabas de hacer dentro del sistema operativo.

---

## 6. Automatización de tareas

### `cron`: tareas que se repiten solas

**cron** es el demonio que ejecuta tareas programadas de forma repetitiva. Cada usuario tiene su propio **crontab**, que se edita con `crontab -e` y se consulta con `crontab -l`. Cada línea sigue esta sintaxis:

```
minuto  hora  día-del-mes  mes  día-de-la-semana  comando
```

Por ejemplo, `0 2 * * *` significa "todos los días a las 2:00 de la madrugada" (el asterisco significa "cualquier valor" en esa posición).

### `at`: una sola vez, en un momento concreto

A diferencia de `cron`, **`at`** programa una tarea para que se ejecute **una única vez**, en un momento futuro que tú indicas: `at 18:00`, escribes el comando (o comandos) y confirmas con Ctrl+D. `atq` lista las tareas pendientes; `atrm <id>` cancela una.

### La trampa más habitual: rutas relativas

Cuando escribes un script para automatizarlo con `cron`, la precaución más importante es usar siempre **rutas absolutas** dentro de él (por ejemplo, `/home/tu_usuario/carpeta` en vez de simplemente `carpeta`). El motivo es que el entorno en el que se ejecuta una tarea de `cron` **no es el mismo** que el de una sesión interactiva de terminal: no hereda automáticamente el mismo directorio de trabajo ni las mismas variables de entorno. Un script con rutas relativas que funciona perfectamente al ejecutarlo a mano puede fallar en silencio cuando lo lanza `cron` — es, con diferencia, la causa más habitual de que "algo que funcionaba deje de funcionar en automático".

Otra buena práctica: redirigir la salida del script a un archivo de registro (`>> archivo.log 2>&1`), para poder comprobar después si la tarea se ejecutó correctamente o falló, y por qué.

### Para practicar

**Actividad:** escribe un script bash sencillo que añada la fecha y hora actuales a un archivo de registro, dale permisos de ejecución y prográmalo con `cron` para que se ejecute cada minuto (solo para poder comprobarlo rápido; en un caso real la frecuencia sería mucho menor).

**Ejemplo resuelto:** script `registro.sh` con el contenido `date >> /home/alumno/registro.log`. Tras `chmod +x registro.sh` y comprobarlo a mano, se añade al crontab la línea `* * * * * /home/alumno/registro.sh`, usando siempre la ruta absoluta del script. Al cabo de dos o tres minutos, `cat /home/alumno/registro.log` muestra varias líneas con fechas distintas.

> **Vamos a practicar: una tarea puntual con `at`**
>
> Programa con `at` una tarea puntual para dentro de 5 minutos (por ejemplo, que cree un archivo con un mensaje concreto), y comprueba que se ejecuta.
>
> **Reflexiona:** si tu script hubiera usado `>> registro.log` en vez de `>> /home/alumno/registro.log`, ¿dónde habría acabado escribiendo el archivo cuando lo ejecutara `cron`, y por qué no sería donde tú esperabas?

---

## Caso práctico integrador

Trabajas en el departamento de soporte de una empresa. Un usuario con Ubuntu te avisa de que, tras una actualización del sistema, el equipo arranca en modo texto en vez de en modo gráfico. Además, te pide dos cosas: que un disco externo que usa para copias de seguridad se monte automáticamente cada vez que enciende el equipo, y que esa copia de seguridad se haga sola cada noche.

Con lo aprendido en esta unidad:

1. El sistema arranca en un target de texto en vez de gráfico. ¿Qué comando usarías para comprobar cuál es el target por defecto, y cuál para corregirlo?
2. Antes de tocar nada en la configuración del sistema, ¿qué medida preventiva deberías haber tomado primero?
3. Quieres que el disco externo se monte automáticamente en `/mnt/backup` en cada arranque. ¿Qué archivo editarías, y con qué dato identificarías el disco de forma fiable, aunque cambie el orden de detección de discos?
4. El usuario quiere la copia de seguridad automática cada noche a las 2:00. ¿Qué herramienta usarías, y qué precaución debes tener con las rutas dentro del script que la realice?
5. Si tras editar la configuración del disco el sistema no llega a arrancar con normalidad, ¿qué harías, paso a paso, para diagnosticarlo y solucionarlo?

---

## Resumen de la unidad

- Tras el gestor de arranque, el kernel arranca systemd (PID 1), que organiza el sistema en unidades y targets (`multi-user.target` en texto, `graphical.target` con interfaz gráfica).
- Una sesión es el conjunto de procesos asociados a un usuario autenticado; cerrar sesión no es lo mismo que apagar el equipo.
- Un entorno de escritorio (GNOME, KDE Plasma, XFCE, MATE...) es la implementación concreta de la capa de shell gráfico; se pueden tener varios instalados a la vez y elegir cuál usar desde la pantalla de login.
- `/etc/fstab` define qué se monta automáticamente en cada arranque; usar UUID en vez del nombre de dispositivo evita problemas si cambia el orden de detección de discos.
- Un error en `fstab` puede impedir el arranque normal, pero es recuperable sin perder datos: el modo de emergencia o el de recuperación de GRUB permiten diagnosticar y corregir.
- `apt update` refresca el índice de paquetes disponibles; `apt upgrade` instala las actualizaciones — no son lo mismo.
- `remove` desinstala un paquete conservando su configuración; `purge` la elimina también. Un `.deb` suelto se instala con `dpkg -i`, resolviendo dependencias rotas con `apt --fix-broken install`.
- DNF y `.rpm` son el equivalente de APT y `.deb` en la familia Fedora/RHEL.
- NetworkManager (`nmcli`/`nmtui`) y los asistentes de dispositivos simplifican configuraciones que también podrían hacerse a mano, reduciendo el margen de error.
- `cron` programa tareas repetitivas; `at`, tareas puntuales de una sola vez. Las rutas dentro de un script programado deben ser siempre absolutas.

## Relación con RA y CE

| CE | Contenido | Dónde se trabaja |
|---|---|---|
| a) | Arranque y parada del sistema, gestión de sesiones | Apartado 1 |
| b) | Interfaces de usuario: línea de comandos frente a entorno gráfico | Apartado 2 |
| c) | Preferencias del entorno personal (incluye varios entornos de escritorio) | Apartado 2 |
| d) | Sistemas de archivos específicos: ext4 y `fstab` | Apartado 3 |
| e) | Recuperación del sistema operativo | Apartado 3 |
| f) | Actualización del sistema operativo | Apartado 4 |
| g) | Instalación/desinstalación de utilidades | Apartado 4 |
| h) | Asistentes de configuración: red y dispositivos | Apartado 5 |
| i) | Automatización de tareas | Apartado 6 |
