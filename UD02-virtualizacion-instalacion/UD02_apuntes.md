# UD02 — Virtualización e instalación

**Módulo:** Sistemas Operativos Monopuesto · 1º SMR

## Introducción

Esta unidad es distinta a UD01: en vez de hablar *sobre* sistemas operativos, vas a instalar y poner en marcha los tuyos propios, sobre máquinas virtuales que vas a seguir usando durante el resto del curso. Todo lo que hagas aquí —tanto si algo sale bien como si tienes que resolver una incidencia— es exactamente el tipo de trabajo que hace un técnico de sistemas en su día a día.

> **Sobre RA2:** parte del contenido de esta unidad (instalación, particionado, licencias, actualización...) refuerza también el Resultado de Aprendizaje RA2, pero **la evaluación oficial de RA2 la hace el tutor o tutora de tu empresa**, no este módulo en el aula. Aquí lo practicas sobre máquina virtual para llegar con soltura al momento de hacerlo en la empresa, sobre hardware real.

Al terminar esta unidad debes ser capaz de:

- Diferenciar una máquina real de una máquina virtual, y explicar qué es un hipervisor y sus dos tipos.
- Justificar cuándo conviene virtualizar y cuándo no.
- Instalar software de virtualización y crear una máquina virtual desde cero.
- Verificar la idoneidad del hardware de un equipo para virtualizar y elegir el sistema operativo adecuado según el caso de uso.
- Elaborar un plan de instalación: particionado (MBR/GPT), configuración de disco y de red de la máquina virtual.
- Instalar sistemas operativos libres y propietarios en máquinas virtuales, incluyendo gestor de arranque y arranque dual.
- Reconocer incidencias típicas de instalación y las normas de utilización de licencias de software.
- Actualizar un sistema recién instalado y configurar los recursos, snapshots y red de una máquina virtual.
- Relacionar la máquina virtual con el sistema operativo anfitrión.
- Realizar pruebas sencillas de rendimiento: tiempo de arranque, velocidad de disco, VM frente a anfitrión.

---

## 1. Virtualización: concepto y tipos de hipervisor

### ¿Qué es una máquina virtual?

Una **máquina virtual** (VM) es un ordenador completo simulado por software: tiene su propia CPU, RAM, disco y tarjeta de red virtuales, y se ejecuta como un programa más dentro de un ordenador físico, al que llamamos **anfitrión** (*host*). Desde dentro de la máquina virtual, el sistema operativo instalado no puede distinguir, salvo por algunas pistas indirectas, que no está funcionando sobre hardware real.

### ¿Por qué existe la virtualización?

La virtualización resuelve varios problemas reales a la vez:

- **Aprovechar mejor el hardware**: un único servidor físico puede alojar decenas de máquinas virtuales, en vez de necesitar un ordenador distinto para cada sistema operativo o servicio.
- **Aislar entornos**: puedes probar software, configuraciones o incluso sistemas operativos nuevos sin arriesgar el sistema principal — si algo sale mal, solo se ve afectada la máquina virtual.
- **Snapshots (instantáneas)**: puedes guardar el estado exacto de una VM en un momento dado y volver a él con un clic, deshaciendo cualquier cambio posterior.
- **Portabilidad**: una máquina virtual es, en esencia, un conjunto de archivos — se puede copiar o mover de un ordenador a otro.
- **Formación y práctica**: puedes instalar, romper y reinstalar sistemas operativos tantas veces como necesites, sin depender de hardware real ni miedo a estropear nada — exactamente el uso que le vas a dar en este módulo.

### El hipervisor: quién reparte el hardware

El **hipervisor** es el software que crea, gestiona y reparte los recursos del hardware físico (CPU, RAM, disco...) entre una o varias máquinas virtuales. Sin hipervisor no hay virtualización posible: es la pieza central de todo el sistema.

### Tipos de hipervisor

Existen dos modelos:

- **Tipo 1 o *bare-metal* (sobre el "metal desnudo")**: se instala directamente sobre el hardware físico, sin ningún sistema operativo anfitrión de por medio. Ejemplos: VMware ESXi, Microsoft Hyper-V (en su modo servidor), Proxmox. Es el modelo que se usa en servidores y centros de datos, porque ofrece el máximo rendimiento posible.
- **Tipo 2 o *hosted* (alojado)**: se instala como una aplicación más, sobre un sistema operativo que ya está en marcha. Ejemplos: VirtualBox, VMware Workstation/Fusion. Es el modelo que vas a usar en este módulo (VirtualBox), porque te permite virtualizar sin dejar de usar tu propio sistema operativo para todo lo demás.

### Ventajas e inconvenientes de virtualizar

**Ventajas:**

- Ahorro de hardware físico (varios sistemas en una sola máquina).
- Aislamiento entre entornos.
- Snapshots para deshacer cambios.
- Portabilidad de un equipo a otro.
- Varios sistemas operativos funcionando a la vez.

**Inconvenientes:**

- Pérdida de rendimiento frente al hardware real: una máquina virtual nunca accede al 100% de la capacidad física del equipo, porque la reparte con el anfitrión (y con otras VMs, si las hay).
- Necesita hardware suficiente para repartir entre el anfitrión y las máquinas virtuales.
- El rendimiento depende de que la CPU soporte virtualización por hardware (Intel VT-x o AMD-V) — prácticamente cualquier procesador de la última década lo soporta, pero conviene saber que existe ese requisito.

### Máquina real frente a máquina virtual

Una máquina virtual no comparte directamente el hardware del anfitrión: tiene su propia CPU, RAM, disco y red virtuales, gestionados por el hipervisor. Lo único que sí depende del hardware físico real es el **rendimiento final** disponible para repartir entre el anfitrión y todas las VMs que tenga en marcha.

### Para practicar

**Actividad:** para cada uno de estos escenarios, indica si conviene usar una máquina real, una máquina virtual con hipervisor tipo 1, o una máquina virtual con hipervisor tipo 2, y justifica tu respuesta:

1. Un centro de datos que aloja los servidores web de cientos de clientes distintos.
2. Un alumno de SMR que quiere probar Linux sin dejar de usar Windows en su portátil personal.
3. Un videojuego exigente que necesita el máximo rendimiento posible de la tarjeta gráfica.
4. Una empresa que quiere que cada empleado pruebe una configuración nueva de software antes de instalarla en todos los equipos de la oficina.

**Ejemplo resuelto (escenario 1):** hipervisor tipo 1 (*bare-metal*) — necesita el máximo rendimiento posible y la capacidad de alojar decenas de máquinas virtuales por servidor físico, sin un sistema operativo anfitrión de por medio consumiendo recursos.

Resuelve el resto por tu cuenta, aplicando el mismo razonamiento: ¿qué pesa más en cada caso, el rendimiento máximo, o el aislamiento sin dejar de usar lo que ya tienes funcionando?

---

## 2. Instalación del software de virtualización

### Qué vamos a usar y por qué

En este módulo vas a usar **VirtualBox** (Oracle), un hipervisor tipo 2: software libre, gratuito para cualquier uso (incluido el comercial), y disponible en Windows, Linux y macOS. Es, con diferencia, el hipervisor más habitual en entornos educativos de FP.

### Antes de instalar: comprobar la virtualización por hardware

Para que las máquinas virtuales funcionen con un rendimiento aceptable, la CPU debe soportar virtualización por hardware (**Intel VT-x** o **AMD-V**) y esa opción debe estar **activada en la BIOS/UEFI**. En Windows 11 puedes comprobarlo sin entrar en la BIOS: Administrador de tareas → pestaña Rendimiento → CPU, donde aparece "Virtualización: Habilitada" o "Deshabilitada". Es la causa más habitual de que, tras instalar VirtualBox sin problemas, las máquinas virtuales no lleguen a arrancar o vayan extremadamente lentas — conviene comprobarlo antes de instalar, no después de que falle.

### Instalación de VirtualBox

Descarga siempre VirtualBox desde su sitio oficial, **virtualbox.org** — nunca de portales de descarga de terceros, por seguridad. Durante la instalación es normal que se interrumpa brevemente la conexión de red: VirtualBox instala sus propios adaptadores de red virtuales, necesarios para que las máquinas virtuales puedan conectarse a internet más adelante.

### El Extension Pack: un componente aparte

El **Extension Pack** añade funciones extra a VirtualBox (soporte de USB 2.0/3.0, arranque por red PXE, cifrado de disco...) y **se instala por separado**, con su propia licencia: la licencia **PUEL**, gratuita para uso personal, educativo y de evaluación, pero de pago para uso comercial. No es lo mismo que VirtualBox en sí (que es software libre bajo licencia GPL, sin esa restricción) — es una distinción que vas a necesitar cuando se trate el tema de licencias de software más adelante en esta unidad.

### La interfaz de VirtualBox

El **Gestor de VirtualBox** es la ventana principal: muestra la lista de máquinas virtuales creadas, y desde ahí se accede a la Configuración de cada una, a sus Snapshots (instantáneas) y a las Herramientas globales (gestor de medios virtuales, configuración de red).

### Una alternativa propietaria: Hyper-V

Si tu equipo tiene Windows Pro o Enterprise, puede tener disponible **Hyper-V**, el hipervisor propietario de Microsoft. A diferencia de VirtualBox, Hyper-V no se "instala" como un programa tradicional: se **activa como característica de Windows** (Panel de control → Programas → Activar o desactivar las características de Windows). Sigue siendo instalar software, solo que empaquetado de otra forma según el fabricante y su modelo de licencia.

**Importante:** VirtualBox y Hyper-V pueden entrar en conflicto si están activos a la vez, porque ambos compiten por el mismo mecanismo de virtualización de hardware de Windows — es habitual tener que desactivar uno para que el otro funcione con normalidad.

### Para practicar

**Actividad:** instala VirtualBox y el Extension Pack en tu equipo, siguiendo el procedimiento visto en clase. Comprueba que el Extension Pack ha quedado instalado correctamente (Archivo → Preferencias → Extensiones). Después, crea una máquina virtual **vacía** (sin instalar ningún sistema operativo todavía) con un nombre identificable — por ejemplo, `TuApellido-UD02-Pruebas` —, indicando solo el tipo y la versión del sistema operativo que instalarás más adelante.

**Por qué importa el nombre:** en cuanto tengas varias máquinas virtuales, un nombre como "VM1" o "prueba" no te va a servir de nada — nombrar bien tus VMs desde el principio es una práctica profesional real, no un capricho de la ficha.

---

## 3. Planificación de la instalación

> Parte de este epígrafe refuerza RA2 (idoneidad de hardware, selección de SO, plan de instalación) — recuerda que la evaluación oficial de esas competencias llega por tu empresa, no por esta unidad.

Instalar sin planificar antes es la causa más habitual de tener que reinstalar desde cero a mitad del proceso. Todo lo que decidas en este epígrafe es lo que vas a seguir al pie de la letra cuando instales de verdad, en el próximo.

### Idoneidad del hardware

Antes de instalar cualquier sistema operativo hay que comparar sus requisitos mínimos (y recomendados) con el hardware disponible — el del equipo real, o el que le vayas a asignar a la máquina virtual.

**Caso especial: Windows 11.** Exige **TPM 2.0**, **Secure Boot** y una CPU de una lista compatible concreta, además de 4 GB de RAM y 64 GB de disco como mínimos. En VirtualBox, si no activas el TPM virtual y el arranque EFI en la configuración de la VM (pestaña Sistema → Placa base → Habilitar EFI; pestaña Seguridad → Habilitar TPM), el instalador de Windows 11 se detiene y no te deja continuar.

Las distribuciones Linux, en comparación, suelen pedir mucho menos — algunas funcionan perfectamente con 1-2 GB de RAM.

> **Curiosidad: se puede instalar Windows 11 sin TPM 2.0 ni Secure Boot**
>
> Microsoft exige estos requisitos de forma oficial, pero existen formas de saltárselos: desde una clave de registro que se añade durante una instalación personalizada, hasta herramientas de creación de unidades de arranque como **Ventoy** (un gestor multi-ISO), combinadas con una imagen de instalación modificada que omite la comprobación. Es una práctica muy habitual en el sector de la reparación y el reacondicionamiento de equipos.
>
> **Con matices:** Microsoft no da soporte oficial a estas instalaciones — el propio fabricante advierte de que el equipo puede no recibir determinadas actualizaciones y queda fuera de la garantía de compatibilidad. Es información que debes conocer como futuro técnico, pero se usa con criterio profesional: informando de las limitaciones reales, no aplicándolo por sistema sin explicar el compromiso que supone.

### Selección del sistema operativo según el caso de uso

No existe "el mejor sistema operativo" en abstracto — depende de para qué se vaya a usar el equipo. Pregúntate:

- ¿Para qué se va a usar el equipo?
- ¿Qué software concreto tiene que poder ejecutar?
- ¿Qué hardware hay disponible?
- ¿Hay presupuesto para licencias, o se necesita algo gratuito?
- ¿Importa que sea código abierto?
- ¿Qué soporte y actualizaciones va a tener a largo plazo?

**Ejemplos:** un servidor web suele elegir Linux (ligero, gratuito, gran soporte de comunidad); un puesto de oficina que depende de software específico de Windows (contabilidad, diseño con herramientas concretas) elige Windows; un equipo con hardware muy limitado se beneficia de una distribución Linux ligera.

### Estilos de partición: MBR frente a GPT

Antes de crear particiones, un disco necesita una **tabla de particiones**: una pequeña estructura, guardada al principio del disco, que dice dónde empieza y dónde termina cada partición. La forma de organizar esa tabla se llama **estilo de partición** (o esquema de particionado), y hoy conviven dos: MBR y GPT. Es una decisión que se toma al preparar el disco, antes de instalar, y que condiciona cómo va a arrancar el equipo.

**MBR (*Master Boot Record*, registro de arranque principal)** es el estilo clásico, creado para los primeros PC de IBM en 1983. Toda la información cabe en el **primer sector del disco** (512 bytes), que contiene a la vez dos cosas: un pequeño programa de arranque y la tabla de particiones, con espacio para solo **cuatro entradas**. De ahí salen sus limitaciones:

- **Máximo 4 particiones primarias.** Para tener más, una de ellas se convierte en **partición extendida**, que hace de "contenedor" de **particiones lógicas**. Funciona, pero es un apaño.
- **Máximo 2 TiB por disco** (con sectores de 512 bytes): la tabla usa números de 32 bits para indicar en qué sector empieza cada partición, y 2³² sectores × 512 bytes = 2 TiB. Un disco de 4 TB con MBR solo puede aprovechar la mitad.
- **Un único punto de fallo:** la tabla está en un solo sector, sin copia. Si ese sector se daña o lo sobrescribe un virus de arranque, el disco parece vacío, aunque los datos sigan ahí.

**GPT (*GUID Partition Table*, tabla de particiones con identificadores únicos)** es el estilo moderno. Nació como parte de la especificación **UEFI**, el firmware que sustituyó a la BIOS, y resuelve todos esos problemas:

- **Hasta 128 particiones** en Windows, todas iguales. No hay primarias, extendidas ni lógicas.
- **Sin límite práctico de tamaño:** usa números de 64 bits, suficientes para discos de miles de millones de terabytes.
- **Más robusto:** guarda una **copia de la tabla al final del disco** y comprueba su integridad con una suma de control (CRC32). Si la tabla principal se daña, se recupera de la copia.
- **Cada partición tiene un identificador único (GUID)** y un tipo, también en forma de GUID, que dice para qué sirve: sistema EFI, datos de Windows, raíz de Linux...
- El primer sector conserva un **MBR de protección** (*protective MBR*): una tabla falsa que dice "todo el disco está ocupado", para que una herramienta antigua que no entienda GPT no crea que el disco está vacío y lo borre.

### MBR, GPT y la forma de arrancar: BIOS frente a UEFI

El estilo de partición no es una decisión aislada: va unido al **firmware** del equipo, el programa grabado en la placa base que se ejecuta nada más encenderlo.

| | BIOS (o UEFI en modo heredado, CSM) | UEFI |
|---|---|---|
| Cómo arranca | Lee el **primer sector** del disco (el MBR) y ejecuta el pequeño programa que hay ahí, que a su vez carga el gestor de arranque | Lee una **partición especial (ESP)** y ejecuta el **archivo** del gestor de arranque que le indica su lista de entradas de arranque |
| Estilo de partición asociado | MBR | GPT |
| Arranque seguro (*Secure Boot*) | No existe | Comprueba que el gestor de arranque está firmado digitalmente, para impedir que arranque software manipulado |

**La regla práctica, en Windows:** Windows instalado en modo UEFI **solo** arranca desde un disco GPT, y en modo BIOS **solo** desde un disco MBR. Linux es más flexible, y puede arrancar desde GPT incluso con BIOS si se crea una pequeña partición especial, pero la combinación habitual también es UEFI + GPT.

De ahí sale el error que vas a ver alguna vez al instalar Windows: *"Windows no se puede instalar en este disco. El disco seleccionado tiene una tabla de particiones MBR. En sistemas EFI, Windows solo se puede instalar en discos GPT."* No significa que el disco esté mal: significa que el equipo arrancó el instalador en modo UEFI y el disco está en MBR. Las soluciones son convertir el disco a GPT (si está vacío, el propio instalador lo hace al borrar todas sus particiones) o, mucho peor, arrancar el instalador en modo heredado.

### Por qué hoy se usa GPT

- **Windows 11 exige UEFI** (además de arranque seguro y TPM 2.0), y con UEFI Windows solo se instala en GPT. En la práctica, **Windows 11 implica GPT**.
- Los fabricantes han ido **eliminando el modo heredado (CSM)** de las placas base: muchos equipos actuales ya no pueden arrancar en modo BIOS.
- Los discos de más de 2 TB son ya habituales, y con MBR no se pueden aprovechar enteros.
- GPT es más robusto (copia de seguridad de la tabla) y más sencillo (sin particiones extendidas ni lógicas).

MBR sigue teniendo sentido en casos concretos: equipos antiguos sin UEFI y algunos pendrives o tarjetas pequeñas que tienen que funcionar en dispositivos antiguos (cámaras, televisores, equipos de música). **Recomendación profesional: GPT siempre, salvo que haya una razón concreta de compatibilidad.**

**Cómo saber qué estilo y qué firmware usa un equipo:**
- **Windows:** `msinfo32` → "Modo de BIOS" (UEFI o Heredado). En **Administración de discos**, clic derecho sobre el disco → **Propiedades** → pestaña **Volúmenes** → "Estilo de partición".
- **Linux:** `sudo fdisk -l` muestra "Disklabel type: gpt" (GPT) o "dos" (MBR). Si existe la carpeta `/sys/firmware/efi`, el sistema ha arrancado en modo UEFI.
- **VirtualBox:** en **Configuración → Sistema → Placa base**, la casilla **Habilitar EFI** decide si la VM usa firmware UEFI o BIOS. Para Windows 11 tiene que estar activada.

**¿Se puede cambiar de MBR a GPT?** Sí, pero con cuidado. La forma sencilla (`convert gpt` en `diskpart`) **solo funciona con el disco vacío**: hay que borrar antes todas las particiones. Windows incluye `mbr2gpt`, que convierte el disco del sistema sin perder datos, pero con condiciones (como máximo tres particiones, entre otras) y obliga a cambiar después el firmware de BIOS a UEFI. Antes de cualquier conversión, copia de seguridad.

### Dónde vive el gestor de arranque en GPT: la partición ESP

Con GPT y UEFI, el gestor de arranque no se guarda "en un sector" (como en MBR), sino como archivos dentro de una partición dedicada llamada **ESP (*EFI System Partition*)**: una partición pequeña (habitualmente entre 100 MB y 1 GB), formateada obligatoriamente en **FAT32** —lo exige la propia especificación UEFI, porque el firmware necesita poder leerla sin ningún sistema operativo de por medio—, identificada por un GUID de tipo de partición en la tabla GPT (no por su nombre, aunque las herramientas la etiquetan como "EFI System Partition").

Dentro contiene una carpeta `/EFI/`, con una subcarpeta por cada sistema operativo instalado: por ejemplo `/EFI/Microsoft/Boot/bootmgfw.efi` (Windows Boot Manager) o `/EFI/ubuntu/grubx64.efi` (GRUB). **Cuando hay dual boot, lo habitual es que ambos sistemas compartan la misma ESP**, cada uno con su propia subcarpeta — el firmware UEFI guarda además, en su memoria NVRAM, una lista de entradas de arranque (visibles con `efibootmgr` en Linux o `bcdedit` en Windows) con un orden de prioridad que decide cuál se arranca por defecto. Vas a necesitar esta idea en el próximo epígrafe, para entender por qué "desaparece" el arranque de Linux tras instalar Windows en dual boot.

**Matiz importante:** la ESP es **por disco**, no por sistema operativo. Si el dual boot se hace en el mismo disco (el caso que vas a practicar en el próximo epígrafe, instalando el segundo sistema en una partición distinta del mismo disco virtual), ambos sistemas comparten esa única ESP. Si en cambio hubiera dos discos distintos, cada uno podría tener su propia ESP, con entradas de arranque independientes. Una vez arrancado el sistema, la ESP no siempre es visible como una unidad más: en Windows aparece oculta, sin letra de unidad; en Linux se suele montar en `/boot/efi` para poder gestionarla.

### Configuración de disco de la máquina virtual

Al crear el disco virtual eliges dos cosas: el **formato** del archivo (VDI, nativo de VirtualBox; VMDK, compatible con VMware; VHD, compatible con Hyper-V) y el **tipo de asignación**: **dinámico** (el archivo crece según se necesita, ocupa menos espacio al principio) o **tamaño fijo** (se reserva todo de golpe, algo mejor de rendimiento). Para las prácticas de este módulo, dinámico es la opción por defecto razonable.

### Configuración de red de la máquina virtual

La tarjeta de red de una VM se puede configurar en varios modos, y cada uno da una visibilidad y conectividad distintas. Lo vas a comprobar con datos reales más adelante en esta unidad, pero conviene tener claros los conceptos ya:

| Modo | Conectividad a internet | Visible desde el anfitrión | Visible entre VMs |
|---|---|---|---|
| **NAT** | Sí | No (por defecto) | No (cada VM aislada) |
| **Red NAT** | Sí | No (por defecto) | Sí, entre VMs de la misma red NAT |
| **Adaptador puente (Bridged)** | Sí | Sí (tiene su propia IP en la red física) | Sí (visible en toda la red local) |
| **Solo anfitrión (Host-only)** | No | Sí | Sí (entre VMs de la misma red host-only) |
| **Red interna (Internal Network)** | No | No | Sí (entre VMs de la misma red interna) |

**NAT** es el modo por defecto: la VM sale a internet a través del anfitrión, como si estuviera detrás de un router, pero queda aislada de todo lo demás. **Adaptador puente** hace que la VM se comporte como un equipo más de la red física, con su propia IP asignada por el mismo router que el resto de dispositivos — el modo que más se parece a tener una máquina real conectada por cable o wifi. **Solo anfitrión** y **Red interna** aíslan completamente de internet, pero permiten que varias VMs se vean entre sí — útiles para practicar redes sin salir a internet ni tocar la red del centro.

### Para practicar

**Actividad:** redacta por escrito el plan de instalación que vas a seguir en el próximo epígrafe, para dos máquinas virtuales (una con una distribución Linux, otra con Windows), indicando para cada una:

1. Sistema operativo elegido y por qué.
2. Requisitos comprobados (RAM, disco, y en el caso de Windows, TPM/Secure Boot).
3. MBR o GPT, y por qué.
4. Esquema de particiones previsto.
5. Tipo y tamaño de disco virtual.
6. Modo de red elegido, y por qué.

**Consejo:** no rellenes esto como un formulario — razona cada elección ahora, con calma, porque cuando tengas el instalador delante en la próxima sesión, ya no va a haber tiempo de pensarlo.

---

## 4. Instalación paso a paso

> Este epígrafe refuerza RA2 (parámetros de instalación, gestor de arranque, incidencias, licencias) — evaluado oficialmente por tu empresa, no por esta unidad. Aquí instalas de verdad dos sistemas operativos completos: uno libre (Linux) y uno propietario (Windows), siguiendo el plan que redactaste en el epígrafe anterior.

### Antes de instalar: descarga de ISOs y verificación de checksums

Descarga siempre las imágenes de instalación (ISO) desde la fuente oficial: la web de la distribución Linux elegida, o el Media Creation Tool / web oficial de Microsoft para Windows. **Comprueba el hash SHA256** de cada ISO antes de usarla, comparándolo con el publicado en la web oficial (`Get-FileHash` en PowerShell, `sha256sum` en Linux). No es un paso opcional: una ISO corrupta o manipulada puede dar problemas serios más adelante — incluido software malicioso, si viene de una fuente no oficial.

### Parámetros básicos de instalación

Casi cualquier instalador va a pedirte lo mismo, así que conviene saber qué esperar: idioma de instalación, distribución de teclado, zona horaria, tipo de instalación (completa o personalizada), partición de destino (aplicando el esquema que decidiste en el epígrafe anterior), cuenta de usuario y contraseña, y nombre del equipo.

### Instalación de la VM Linux

Arranca la VM desde la ISO montada como unidad óptica virtual y sigue el asistente: idioma, teclado, tipo de instalación (todo el disco, o particionado manual siguiendo tu plan), creación del usuario, copia de archivos, reinicio y primer arranque. Al terminar, **instala las Guest Additions de VirtualBox** — mejoran la resolución de pantalla y añaden carpetas compartidas y portapapeles compartido con tu equipo anfitrión. Lo verás en detalle en el próximo epígrafe, pero conviene dejarlas instaladas ya para que la VM sea cómoda de usar desde el principio.

### Instalación de la VM Windows

Mismo procedimiento, con sus particularidades: arrancar desde la ISO, elegir idioma y edición, aceptar la licencia, tipo de instalación (personalizada, para aplicar el particionado GPT que planificaste), pantalla de clave de producto (puedes continuar sin clave para fines educativos — el sistema queda sin activar hasta introducir una válida), tipo de cuenta (**usa cuenta local**, no cuenta Microsoft, para evitar vincular datos personales a un sistema que vas a usar en el aula), configuración de privacidad, primer arranque. Instala también las Guest Additions al terminar.

### Cómo crear una cuenta local en Windows 11

Las versiones recientes de Windows 11 ocultan la opción de **cuenta local** (una cuenta que solo existe en ese equipo) y empujan a iniciar sesión con una **cuenta Microsoft** (vinculada a un correo y sincronizada en la nube) durante la configuración inicial, la fase llamada **OOBE** (*Out-Of-Box Experience*, las pantallas azules que aparecen tras la copia de archivos). No es una limitación técnica, sino una decisión comercial de Microsoft para fomentar su ecosistema: OneDrive, sincronización, Microsoft 365...

En el aula vas a usar **cuenta local**: no tiene sentido vincular tu cuenta personal a una VM de prácticas, y en una empresa es lo habitual en equipos que se gestionan de forma centralizada. Estas son las formas de conseguirla, de la más directa a la menos:

1. **El comando `start ms-cxh:localonly`.** Cuando el asistente te pida iniciar sesión con una cuenta Microsoft, pulsa **Mayús + F10**. Se abre una ventana de símbolo del sistema. Escribe `start ms-cxh:localonly` y pulsa Intro: aparece directamente la ventana **"Crear un usuario para este equipo"**, en la que introduces el nombre de usuario, la contraseña y las tres preguntas de seguridad. Al terminar, la instalación continúa con esa cuenta local.
2. **Sin conexión de red.** Si desconectas el adaptador de red de la VM antes de llegar a la pantalla "Vamos a conectarte a una red", en algunas versiones aparece **"No tengo internet" → "Continuar con configuración limitada"**, que crea una cuenta local.
3. **El comando `oobe\bypassnro`** (Mayús + F10 en la pantalla de red). Era el método clásico, pero Microsoft lo ha retirado de las versiones más recientes de Windows 11. Si lo encuentras en un tutorial, ya sabes por qué puede no funcionarte.

**Aviso:** Microsoft cambia este comportamiento con frecuencia entre versiones, y ha anunciado que irá cerrando estos atajos, incluido `ms-cxh:localonly`. Que un método funcione o no depende de la compilación exacta de tu ISO. Si ninguno funciona, no uses una cuenta Microsoft personal: la solución oficial, que no depende de estos atajos, es el **archivo de respuestas** del apartado especial que viene a continuación. Es, en sí mismo, un ejemplo real de **incidencia de instalación**: lo que funciona hoy puede dejar de funcionar dentro de unos meses, y por eso un técnico documenta siempre qué método ha usado y con qué versión.

### Apartado especial: instalación desatendida de Windows 11

Hasta ahora has instalado Windows **pantalla a pantalla**, respondiendo cada pregunta del asistente. Para una VM de prácticas está bien. Pero piensa en un técnico que tiene que preparar las 30 máquinas de un aula, o los 200 portátiles de una empresa: hacerlo a mano llevaría días, y además cada equipo acabaría configurado de forma ligeramente distinta según quién lo instalara.

La solución es la **instalación desatendida** (*unattended installation*): una instalación en la que las respuestas a las preguntas del asistente se dan **por adelantado, en un archivo**, y el programa de instalación las aplica solo, sin que nadie esté delante. Es como se despliega Windows en el mundo profesional, y también lo que resuelve de forma limpia y oficial el problema de la cuenta Microsoft del apartado anterior.

#### Cómo funciona: el archivo de respuestas

Ese archivo se llama **archivo de respuestas** (*answer file*). Es un archivo de texto en formato **XML**: un formato en el que la información se organiza en **etiquetas** que se abren y se cierran, `<Etiqueta>valor</Etiqueta>`, y que pueden ir unas dentro de otras.

Lo que hace que funcione sin tocar nada es su **nombre**. Cuando arranca, el programa de instalación de Windows busca un archivo llamado exactamente **`autounattend.xml`** en la **raíz de todas las unidades extraíbles** del equipo (USB, DVD, otras unidades ópticas de la VM). Si lo encuentra, lo usa.

La instalación de Windows pasa por varias **fases** (*configuration passes*), y el archivo de respuestas puede dar órdenes para cada una por separado. Las tres que más te interesan:

| Fase | Cuándo ocurre | Ejemplos de lo que se puede automatizar |
|---|---|---|
| `windowsPE` | Al principio, desde la ISO, antes de copiar Windows | Idioma, edición, clave de producto, **particionado del disco** |
| `specialize` | En el primer arranque, ya desde el disco | Nombre del equipo, unirse a un dominio, configuración de red |
| `oobeSystem` | La **configuración inicial** (las pantallas azules del final) | **Cuentas de usuario**, zona horaria, privacidad, saltar la cuenta Microsoft |

Un archivo de respuestas no tiene por qué automatizarlo todo: **puede contener solo una fase**. El que vas a usar en clase solo tiene `oobeSystem`, así que el idioma, la edición y el particionado los sigues haciendo tú a mano, y solo se automatiza la parte de la cuenta.

#### Un archivo de respuestas mínimo, explicado

Este es el archivo que te facilitará tu profesor o profesora (sin los comentarios):

```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend"
          xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State">
  <settings pass="oobeSystem">
    <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64"
               publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
      <OOBE>
        <HideOnlineAccountScreens>true</HideOnlineAccountScreens>
        <HideWirelessSetupInOOBE>true</HideWirelessSetupInOOBE>
        <HideEULAPage>true</HideEULAPage>
        <ProtectYourPC>3</ProtectYourPC>
      </OOBE>
      <UserAccounts>
        <LocalAccounts>
          <LocalAccount wcm:action="add">
            <Name>alumno</Name>
            <DisplayName>Alumno SOM</DisplayName>
            <Group>Administradores</Group>
            <Password>
              <Value>Som2026!</Value>
              <PlainText>true</PlainText>
            </Password>
          </LocalAccount>
        </LocalAccounts>
      </UserAccounts>
      <TimeZone>Romance Standard Time</TimeZone>
    </component>
  </settings>
</unattend>
```

Leído de fuera hacia dentro:

- **`<unattend>`** es la etiqueta raíz de todo archivo de respuestas.
- **`<settings pass="oobeSystem">`** indica que lo de dentro se aplica en la fase de configuración inicial.
- **`<component name="Microsoft-Windows-Shell-Setup" ...>`** dice a qué parte de Windows van dirigidos los ajustes. Los atributos de esta línea son siempre iguales para Windows de 64 bits. Lo único que cambia de un equipo a otro es `processorArchitecture`: `amd64` para procesadores Intel/AMD y `arm64` para equipos ARM.
- **`<OOBE>`** agrupa las pantallas de la configuración inicial:
  - `HideOnlineAccountScreens` oculta las pantallas de cuenta Microsoft;
  - `HideWirelessSetupInOOBE`, la de conectarse a una red wifi;
  - `HideEULAPage`, el contrato de licencia (ya aceptado al instalar);
  - `ProtectYourPC` con el valor `3`, las preguntas de privacidad (no aplica la configuración "recomendada" de envío de datos).
- **`<LocalAccount>`** crea una **cuenta local** con su nombre de usuario (`Name`), el nombre que se muestra (`DisplayName`), el **grupo** al que pertenece y su contraseña. El grupo se escribe **en el idioma de Windows**: `Administradores` en un Windows en español, `Administrators` en uno en inglés.
- **`<TimeZone>`** fija la zona horaria. `Romance Standard Time` es el nombre interno de la zona de Madrid.

**Fíjate en la contraseña:** está escrita **en texto plano** (`PlainText` a `true`). Algunas herramientas la guardan "codificada" en Base64, pero eso no es cifrado: cualquiera puede descodificarla en segundos. Un archivo de respuestas es, por tanto, un **documento sensible**. En una empresa se protege como cualquier otra credencial, y después de instalar se cambia la contraseña.

#### Opción 1: escribir o generar el archivo tú mismo

- **A mano**, partiendo de un ejemplo como el anterior. Es la mejor forma de entender qué hace cada línea, pero un error de una sola letra en una etiqueta hace que el programa de instalación ignore el ajuste o se detenga con un error.
- **Con el Administrador de imágenes de sistema de Windows** (*Windows System Image Manager*, WSIM): la herramienta oficial de Microsoft, incluida en el **Windows ADK** (*Assessment and Deployment Kit*). Abre la imagen de Windows, muestra todos los ajustes posibles organizados por fase y componente, y comprueba que el archivo es válido. Es la que se usa en entornos profesionales.
- **Con un generador web**, como el *Unattend Generator* de Christoph Schneegans (schneegans.de): un formulario en el que marcas opciones (idioma, cuentas, particionado, aplicaciones que quitar...) y te descarga el `autounattend.xml`. Es muy cómodo, pero es una web de terceros: **no escribas contraseñas reales** en ella, usa una provisional y cámbiala después.

#### Opción 2: herramientas que lo preparan por ti

- **Rufus**, el programa más usado para crear USB de instalación, pregunta al crear un USB de Windows 11 si quieres **eliminar el requisito de cuenta Microsoft en línea** y **crear una cuenta local** con un nombre de usuario. Lo que hace por dentro es escribir un archivo de respuestas en el USB. Sirve para equipos reales.
- **La instalación desatendida de VirtualBox**, la que en la Práctica 2 te pedimos desactivar, también genera su propio archivo de respuestas, con el usuario y la contraseña que indicas en el asistente. Su inconveniente para aprender es que automatiza **toda** la instalación, también el particionado.

#### Cómo se usa en una VM de VirtualBox

El archivo tiene que estar en la raíz de una unidad extraíble. En una VM, lo más sencillo es meterlo en una **ISO pequeña** y montarla como **segunda unidad óptica**:

1. Con la VM apagada, **Configuración → Almacenamiento** → en el controlador, **Añadir unidad óptica** → elige la ISO del archivo de respuestas. La ISO de Windows 11 sigue en la primera unidad, que es desde la que arranca la VM.
2. Instala con normalidad. Al llegar a la configuración inicial, las pantallas de cuenta Microsoft, wifi y privacidad no aparecen, y Windows inicia sesión directamente con la cuenta local del archivo.
3. Comprueba en una terminal con `whoami` (el usuario con el que has entrado) y con `net user` (las cuentas locales del equipo).
4. Quita la segunda unidad óptica y cambia la contraseña.

Para crear tu propia ISO con un archivo de respuestas, desde Ubuntu basta con un comando: `genisoimage -J -r -V RESPUESTAS -o respuestas.iso carpeta/`, donde `carpeta/` contiene el `autounattend.xml`. En un equipo real, basta con copiar el archivo a la raíz del USB de instalación de Windows.

> **Vamos a practicar (opcional): una instalación de Windows 11 sin la pantalla de cuenta**
>
> **Paso a paso:**
>
> 1. Crea una VM nueva y temporal, `Apellido-UD02-Desatendida`, con la ISO de Windows 11 y la instalación desatendida **de VirtualBox desactivada**, igual que en la Práctica 2.
> 2. Añádele una segunda unidad óptica con la ISO del archivo de respuestas que te facilitará tu profesor o profesora.
> 3. Instala Windows haciendo a mano el particionado, y observa qué pantallas de la configuración inicial ya no aparecen.
> 4. Cuando termine, ejecuta `whoami` y `net user` y comprueba que la cuenta `alumno` existe y pertenece a los administradores (`net localgroup Administradores`).
> 5. Elimina la VM con **Eliminar todos los archivos**: ocupa más de 20 GB.
>
> **Reflexiona:** en el paso 3 el particionado lo has hecho tú, pero la cuenta no. ¿Por qué? ¿Qué tendrías que añadir al archivo para que tampoco te preguntara por el particionado, y en qué fase iría? Y si tuvieras que preparar las 30 VMs del aula, ¿qué ventajas tendría usar este archivo frente a hacerlo a mano en cada una?

### El gestor de arranque: GRUB, Windows Boot Manager y dual boot

Un **gestor de arranque** (*bootloader*) es el software que se ejecuta justo después del firmware (BIOS/UEFI) y que carga el sistema operativo — si hay varios sistemas instalados, decide (o pregunta) cuál arrancar. **GRUB** es el gestor más habitual en Linux; **Windows Boot Manager** cumple el mismo papel en Windows.

**Dual boot** es instalar dos sistemas operativos en el mismo disco, eligiendo cuál arrancar cada vez desde el menú del gestor de arranque. Lo vas a comprobar de forma segura **dentro de una VM**: instalando un segundo sistema operativo en la misma VM ya creada, en una partición distinta del mismo disco virtual, y viendo aparecer el menú de selección al arrancar.

**Algo que te va a pasar de verdad:** instalar Windows *después* de Linux en dual boot suele hacer "desaparecer" el arranque de Linux. La causa exacta, con GPT/UEFI, no es que Windows "sobrescriba un sector" (esa es la lógica de MBR, no de GPT): ¿recuerdas la ESP del epígrafe anterior? Ambos sistemas comparten esa misma partición, cada uno con su propia subcarpeta de archivos `.efi`. Lo que hace el instalador de Windows es **añadir su propia entrada de arranque y ponerse el primero en el orden de prioridad de la NVRAM del firmware**, dejando la entrada de GRUB intacta pero relegada. Por eso "parece" que Linux ha desaparecido, aunque sus archivos siguen ahí. No es un fallo tuyo: es un comportamiento esperado del instalador de Windows, que asume que va a ser el único sistema operativo del equipo.

### Incidencias típicas de instalación

| Síntoma | Causa probable | Solución |
|---|---|---|
| El instalador no arranca desde el USB/ISO | Orden de arranque incorrecto en la BIOS/UEFI, o imagen corrupta/mal grabada | Comprobar el checksum de la ISO; revisar el orden de arranque |
| No hay espacio suficiente para crear la partición | Disco demasiado pequeño, o particiones mal planificadas | Redimensionar el disco virtual o revisar el plan |
| Tras dual boot, ya no aparece la opción de arrancar Linux | Windows ha añadido su propia entrada de arranque en la NVRAM y se ha puesto primero en el orden de prioridad (los archivos de GRUB en la ESP suelen seguir intactos) | Reordenar/recuperar la entrada de arranque con `efibootmgr` desde un live USB de Linux, o repararla con una herramienta como Boot-Repair |
| La instalación se queda colgada o va muy lenta | Recursos insuficientes asignados a la VM (RAM/CPU) | Revisar y aumentar la configuración de recursos de la VM |
| Tras instalar, no hay conexión a internet | Modo de red mal configurado, o faltan las Guest Additions | Revisar el modo de red; reinstalar Guest Additions |

### Licencias de software

**Software propietario/comercial y el EULA.** El fabricante retiene los derechos; tú adquieres una licencia de uso, no la propiedad del software. El **EULA** (*End User License Agreement*, contrato de licencia de usuario final) es el texto que aceptas al instalar este tipo de software, y es un término que se usa casi en exclusiva para el **software propietario**: define qué puedes y qué no puedes hacer con tu copia (en cuántos equipos, si se puede transferir, si se permite modificar o descompilar el programa...). Ojo: "EULA" no es sinónimo de "cualquier licencia" — el software libre también tiene licencia, pero casi nunca se le llama EULA (verás por qué en un momento).

Dentro del software propietario/comercial hay tres modelos de licencia distintos, con consecuencias muy prácticas:

- **OEM** (*Original Equipment Manufacturer*): vinculada a una pieza de hardware concreta (normalmente la placa base) desde su primera activación. Más barata, pero **no se puede transferir a otro equipo**. Es la que suele venir preinstalada en equipos nuevos de fabricantes, o la que compran los integradores de sistemas para venderla junto con el hardware.
- **Retail** (de caja/minorista): comprada de forma independiente, sin vincularse a ningún hardware. Se puede desactivar en un equipo y activar en otro — más cara que OEM, a cambio de esa flexibilidad.
- **Por volumen**: acuerdos especiales para licenciar software en muchos equipos a la vez (empresas, centros educativos), en vez de una licencia por equipo.

**Software libre (FOSS) y las cuatro libertades.** El software libre también tiene licencia (GPL, MIT, Apache...) — es un contrato legal igual que un EULA, pero con el propósito opuesto: en vez de restringir, **otorga** libertades. Por eso, aunque técnicamente también es un acuerdo de licencia, casi nunca se le llama EULA. La Free Software Foundation define el software libre mediante **cuatro libertades**, numeradas de 0 a 3:

- **Libertad 0:** usar el programa con cualquier propósito.
- **Libertad 1:** estudiar cómo funciona y modificarlo (requiere acceso al código fuente).
- **Libertad 2:** redistribuir copias para ayudar a otros.
- **Libertad 3:** distribuir copias de tus versiones modificadas (también requiere acceso al código fuente).

Son estas libertades las que distinguen software libre de *freeware*: un programa puede ser gratuito sin ser libre. Dentro del software libre hay además enfoques distintos: licencias **copyleft** como la GPL exigen que las versiones modificadas y redistribuidas también sean libres; licencias **permisivas** como MIT o Apache permiten incluso incorporar ese código en software propietario, sin esa obligación.

**Freeware.** Software gratuito, pero sin las libertades del software libre: el código fuente no es accesible, y suele tener su propio EULA restrictivo pese a costar 0 € (puede prohibir la redistribución, la ingeniería inversa, el uso comercial...). Se confunde fácilmente con software libre por ser gratis, pero la clave no es el precio: son las libertades que otorga.

**Activación con clave de producto:** mecanismo habitual del software propietario (como Windows) para verificar que tu copia corresponde a una licencia válida, y para vincularla a su modelo concreto (OEM/Retail/volumen).

¿Recuerdas el **Extension Pack de VirtualBox** del epígrafe 2? Licencia PUEL: gratuita para uso personal y educativo, de pago para uso comercial — un caso real que no encaja del todo en ninguna de las categorías anteriores por sí solo.

### Para practicar

**Actividad:** siguiendo el plan que redactaste en el epígrafe anterior, instala una VM con la distribución Linux elegida y otra con Windows, comprobando el checksum de cada ISO antes de empezar. Sobre la VM Linux ya instalada, instala un segundo sistema operativo en una partición distinta del mismo disco virtual, para ver en vivo el menú de selección de GRUB (dual boot).

**Documenta cualquier incidencia real** que te encuentres durante el proceso, con este formato: síntoma → causa probable → solución aplicada. No pasa nada si algo falla — es exactamente el tipo de situación con la que te vas a encontrar como técnico, y saber documentarla bien vale más que una instalación perfecta a la primera sin entender del todo cada paso.

---

## 5. Post-instalación y configuración de la VM

> Refuerza RA2 (actualización) y cubre RA5 (recursos, snapshots, red, relación con el anfitrión). Tus dos VMs ya están instaladas — ahora toca dejarlas listas para usarlas el resto del curso.

### Actualización post-instalación

En Windows: Configuración → Windows Update, instala todas las actualizaciones pendientes y reinicia cuantas veces haga falta. En Linux: `sudo apt update && sudo apt upgrade` (Debian/Ubuntu) o `sudo dnf upgrade` (Fedora). Hazlo justo después de instalar: parches de seguridad, controladores actualizados, estabilidad — un sistema recién instalado nunca es el más seguro posible hasta que se actualiza.

### Snapshots: qué son y buenas prácticas

Un **snapshot** (instantánea) guarda el estado exacto de una VM en un momento dado —disco, memoria RAM y configuración— y te permite volver a ese punto con un clic, deshaciendo cualquier cambio posterior. Se crean desde el Gestor de VirtualBox (botón derecho sobre la VM → Instantánea → Tomar). Ponles siempre un nombre y una descripción que expliquen *por qué* la tomaste — no "Snapshot 1", sino "Antes de actualizar" o "Sistema recién instalado, sin configurar".

**Cuándo tomar una:** antes de cualquier cambio arriesgado — actualizar el sistema, instalar software nuevo, cambiar una configuración importante. Si algo sale mal, restauras la instantánea anterior en segundos, en vez de reinstalar desde cero.

**Aviso importante (¿recuerdas el journaling de UD01?):** un snapshot **no es una copia de seguridad real**. Si borras la VM entera (o pierdes el archivo de su disco virtual), pierdes también todas sus instantáneas — viven dentro del mismo conjunto de archivos de la VM, no en un lugar separado. Para una copia de seguridad de verdad hace falta exportar la VM completa a otro disco o servicio distinto.

### Ajustar los recursos de la VM después de instalada

A diferencia de cuando creaste la VM (donde asignaste RAM/CPU antes de instalar nada), ahora puedes ajustarlos con el sistema ya en marcha, desde Configuración de la VM (con la VM apagada). Tiene sentido hacerlo si el sistema va lento, o si le asignaste más recursos de los que realmente necesita y quieres liberarlos para tu equipo anfitrión.

### Repaso de los modos de red: práctica de comparación real

Recupera la tabla de modos de red del epígrafe 3 y, con tus dos VMs ya instaladas, comprueba **con datos reales** lo que hasta ahora solo habías visto en teoría:

1. Configura cada VM en un modo de red distinto (empieza por NAT en ambas, el modo por defecto).
2. En cada VM, obtén su dirección IP: `ipconfig` en Windows, `ip addr` (o `ifconfig`) en Linux.
3. Comprueba si una VM hace `ping` a la otra.
4. Comprueba si cada VM tiene salida a internet (`ping 8.8.8.8`, o abre un navegador).
5. Repite cambiando el modo de red de ambas VMs a **Red interna**, y después a **Adaptador puente** — anota en cada caso qué IP obtiene cada una, si se ven entre sí, y si tienen internet.
6. Contrasta lo que has obtenido con lo que predecía la tabla teórica del epígrafe 3. Si algo no coincide, no lo apuntes sin más: averigua por qué.

### Relación VM-anfitrión: Guest Additions y carpetas compartidas

Las **Guest Additions**, que ya instalaste en el epígrafe anterior, son un conjunto de controladores y utilidades que mejoran la integración de la VM con tu equipo anfitrión: resolución de pantalla ajustable automáticamente, portapapeles compartido, arrastrar y soltar archivos entre ambos, y **carpetas compartidas**.

Una **carpeta compartida** es una carpeta de tu equipo anfitrión a la que la VM accede como si fuera una unidad de red propia, sin pasar por la tarjeta de red virtual —es un mecanismo distinto, gestionado directamente por el hipervisor—, útil para pasar archivos entre tu equipo real y la VM sin depender de la configuración de red. Se configura desde Configuración de la VM → Carpetas compartidas.

### Para practicar

**Actividad:**

1. Actualiza completamente ambas VMs y toma una instantánea con nombre "Post-actualización" en cada una al terminar.
2. Realiza la práctica de comparación de modos de red descrita arriba, probando al menos tres modos distintos, y completa una tabla con: modo de red, IP obtenida por cada VM, si hacen ping entre sí, si tienen salida a internet.
3. Configura una carpeta compartida entre tu equipo anfitrión y una de las VMs, y copia un archivo de prueba en ambos sentidos.

---

## 6. Pruebas de rendimiento y cierre

Hoy se cierra la unidad: toca medir con datos reales lo que hasta ahora solo se ha argumentado en teoría, y hacer una primera toma de contacto con una tecnología muy presente en el sector: los contenedores.

### Pruebas de rendimiento básicas

**Aviso:** aquí solo se miden benchmarks sencillos y puntuales — no se entra en herramientas de monitorización de procesos o servicios en marcha con detalle (eso llega con RA4).

- **Tiempo de arranque:** cronometra cuánto tarda en arrancar cada VM (desde "Iniciar" hasta que el sistema está listo) y compáralo con el tiempo de arranque de tu propio equipo anfitrión.
- **Velocidad de disco:** una prueba simple de lectura/escritura — `CrystalDiskMark` en Windows, o `dd` en Linux (`dd if=/dev/zero of=testfile bs=1M count=1024; sync`) — comparando el resultado dentro de la VM frente al mismo disco medido en el anfitrión.
- **Interpreta el resultado, no solo lo anotes:** la VM va a rendir algo menos en ambas pruebas, casi siempre — es la confirmación con datos reales de lo que viste en el epígrafe 1: el hipervisor reparte el mismo hardware físico entre el anfitrión y la VM, así que ninguno de los dos accede al 100% de su capacidad.

### Ampliación: introducción a Docker

Un **contenedor** es una forma distinta —y más ligera— de aislar una aplicación y sus dependencias, sin virtualizar un hardware completo: a diferencia de una máquina virtual, que incluye su propio sistema operativo entero (con su propio núcleo), un contenedor **comparte el núcleo del sistema anfitrión**, y solo empaqueta la aplicación junto con lo que necesita para funcionar igual en cualquier equipo. Esto lo hace mucho más ligero y rápido de arrancar que una VM.

**Docker** es la herramienta de contenedores más extendida hoy. No la vas a instalar ni practicar en esta unidad —es solo una introducción conceptual—, pero conviene que sepas que existe y en qué se diferencia de lo que acabas de aprender: es tecnología muy presente en el sector, y es probable que la veas con más detalle más adelante en el ciclo.

### Caso práctico integrador

Un cliente de la empresa donde haces la FP Dual te pide preparar un equipo con dos sistemas operativos (Windows y Linux), para que dos departamentos distintos puedan usarlo según necesiten sin arriesgar los datos de uno si el otro falla. Además, quiere que ambos sistemas puedan verse entre sí en red para compartir archivos, pero sin quedar expuestos directamente a Internet más de lo necesario. Con lo aprendido en esta unidad:

1. ¿Instalarías esto como dual boot en un único equipo físico, o como dos máquinas virtuales independientes? Justifica con lo visto en el epígrafe 1.
2. ¿Qué sistema de particionado usarías (MBR o GPT) y por qué?
3. ¿Qué modo de red elegirías para que ambos sistemas se vean entre sí sin quedar expuestos directamente a Internet?
4. Antes de entregar el equipo, ¿qué comprobarías respecto a actualizaciones y licencias?
5. ¿Qué medida tomarías antes de hacer cualquier cambio arriesgado en un sistema ya configurado y entregado?

### Para practicar

**Actividad:** mide el tiempo de arranque y la velocidad de disco (lectura/escritura) de tus dos VMs y de tu equipo anfitrión, y complétalo en una tabla comparativa. Después, resuelve por escrito el caso práctico integrador de arriba.

**Ejemplo resuelto (pregunta 2 del caso):** GPT, salvo que haya una razón concreta de compatibilidad con hardware muy antiguo — es el estándar actual y obligatorio si en algún momento se necesita Windows con Secure Boot.

Resuelve el resto por tu cuenta, aplicando lo visto en cada epígrafe correspondiente.

---

## Resumen de la unidad

- Una máquina virtual es un ordenador completo simulado por software, gestionado por un hipervisor (tipo 1, sobre el hardware directamente; tipo 2, sobre un sistema operativo ya en marcha).
- Virtualizar tiene ventajas reales (aislamiento, snapshots, portabilidad, aprovechamiento del hardware) e inconvenientes reales (pérdida de rendimiento) — la decisión depende del caso de uso.
- Antes de instalar cualquier sistema hay que planificar: idoneidad del hardware, selección del SO, MBR o GPT (con la partición ESP en GPT/UEFI), formato de disco virtual y modo de red.
- La instalación sigue unos parámetros básicos comunes, con particularidades de cada fabricante (activación de Windows, cuentas Microsoft frente a locales).
- El gestor de arranque (GRUB, Windows Boot Manager) decide qué sistema arranca; en dual boot sobre GPT/UEFI, ambos sistemas comparten la misma ESP y compiten por la prioridad de arranque en la NVRAM, no por "un sector".
- Las incidencias de instalación son parte normal del oficio — lo importante es diagnosticar con método, no memorizar soluciones.
- El software tiene distintos modelos de licencia (propietario con EULA, OEM, Retail, por volumen; software libre con las cuatro libertades; freeware) que no deben confundirse entre sí.
- Tras instalar, hay que actualizar el sistema, tomar snapshots antes de cambios arriesgados (sin confundirlos con copias de seguridad reales) y configurar la red y la relación con el anfitrión según lo que se necesite.
- Los modos de red de una VM dan visibilidad y conectividad distintas — la elección debe basarse en el caso de uso real.
- Las pruebas de rendimiento confirman con datos reales que una VM rinde algo menos que el hardware real.
- Los contenedores (Docker) son una alternativa más ligera a la virtualización completa, que comparte el núcleo del anfitrión — relevante en el sector, aunque fuera del contenido evaluable de esta unidad.

## Relación con RA y CE

| CE | Contenido | Dónde se trabaja |
|---|---|---|
| RA5-a | Máquina real vs. virtual, tipos de hipervisor | Apartado 1 |
| RA5-b | Ventajas e inconvenientes de virtualizar | Apartado 1 |
| RA5-c | Instalación del software de virtualización | Apartado 2 |
| RA2-a *(refuerzo)* | Idoneidad del hardware | Apartado 3 |
| RA2-b *(refuerzo)* | Selección del sistema operativo | Apartado 3 |
| RA2-c + RA5-d | Plan de instalación (particionado, disco, red) | Apartado 3 |
| RA2-d *(refuerzo)* | Parámetros básicos de instalación | Apartado 4 |
| RA2-e *(refuerzo)* | Gestor de arranque, dual boot | Apartado 4 |
| RA2-f *(refuerzo)* | Incidencias de instalación | Apartado 4 |
| RA2-g *(refuerzo)* | Licencias de software | Apartado 4 |
| RA2-h *(refuerzo)* | Actualización post-instalación | Apartado 5 |
| RA5-e | Configuración de la VM (recursos, snapshots, red) | Apartado 5 |
| RA5-f | Relación VM-anfitrión | Apartado 5 |
| RA5-g | Pruebas de rendimiento | Apartado 6 |

> Recuerda: los CE de RA2 marcados como "refuerzo" se evalúan oficialmente en tu empresa, durante la fase de FP Dual (mayo–10 junio) — aquí los practicas sobre máquina virtual para llegar con soltura a ese momento.

---

*(UD02 completa: 6 epígrafes desarrollados.)*
