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

### Particionado: MBR frente a GPT

**MBR (Master Boot Record)** es el esquema de particionado tradicional, en uso desde los años 80: máximo 4 particiones primarias (o 3 primarias + 1 extendida, que puede contener varias particiones lógicas dentro), límite de 2 TiB por disco, arranque mediante BIOS tradicional.

**GPT (GUID Partition Table)** es el esquema moderno: hasta 128 particiones sin necesidad de particiones extendidas, sin el límite práctico de 2 TiB de MBR, y con una copia de seguridad de su propia tabla de particiones al final del disco (más resistente a corrupción). Requiere **UEFI** para arrancar desde él con todas sus ventajas.

**Por qué importa esta decisión:** Windows 11, con Secure Boot activado, **exige GPT** — no se puede instalar sobre un disco en MBR. Linux funciona perfectamente con cualquiera de los dos. Recomendación práctica: usa **GPT siempre que sea posible**, salvo que necesites compatibilidad específica con sistemas muy antiguos sin UEFI.

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

Las versiones más recientes de Windows 11 ocultan deliberadamente la opción de cuenta local y prácticamente obligan a iniciar sesión con una cuenta Microsoft durante la configuración inicial. No es una limitación técnica: es una decisión comercial de Microsoft para fomentar el uso de su ecosistema (OneDrive, sincronización en la nube, Microsoft 365...). Dos formas de conseguir una cuenta local:

1. **Sin conexión de red.** En la pantalla "Vamos a conectarte a una red", si no hay ninguna red disponible (o desconectas el adaptador de red de la VM antes de llegar aquí), a veces aparece "No tengo internet" → "Continuar con configuración limitada", que sí crea una cuenta local. En versiones muy recientes puede no aparecer a la primera.
2. **El comando `bypassnro`.** En esa misma pantalla, pulsa **Mayús + F10** para abrir una ventana de símbolo del sistema, escribe `oobe\bypassnro` y pulsa Intro. El equipo se reinicia y retoma la configuración saltándose el requisito de conexión obligatoria, recuperando la opción de continuar sin red.

**Aviso:** Microsoft cambia este comportamiento entre versiones de Windows con cierta frecuencia — el método exacto puede variar según la build que estés instalando. Es, en sí mismo, un buen ejemplo real de "incidencia de instalación": lo que funciona hoy puede no funcionar igual dentro de unos meses.

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
