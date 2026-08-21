# UD01 — Fundamentos del sistema operativo

**Módulo:** Sistemas Operativos Monopuesto · 1º SMR

## Introducción

Antes de instalar un sistema operativo, configurarlo o administrarlo, necesitas entender qué es y cómo está organizado por dentro. Esta unidad te da el vocabulario y los conceptos base que vas a usar constantemente a lo largo de todo el curso: cuando dentro de unas semanas instales tu primera máquina virtual, cuando particiones un disco, cuando ejecutes tu primer comando `chmod`, vas a estar aplicando directamente lo que aprendas aquí.

Dos contenidos de esta unidad merecen especial atención porque los vas a usar de forma literal y constante a partir de ahora: **el sistema de archivos** (cómo se organiza la información en un disco) y **los permisos** (quién puede hacer qué con cada archivo). Por eso ocupan más de la mitad del peso de esta unidad — no porque sean "más difíciles", sino porque son los que de verdad vas a necesitar desde el primer día de prácticas con máquinas virtuales.

Al terminar esta unidad debes ser capaz de:

- Explicar qué es un sistema informático y dónde encaja el sistema operativo dentro de él.
- Describir qué hace un sistema operativo y cómo se organiza por capas.
- Distinguir un programa de un proceso, y nombrar los estados básicos de un proceso.
- Moverte conceptualmente por la jerarquía de archivos de Windows y de Linux.
- Convertir entre binario, octal y decimal para leer y escribir permisos, y calcular múltiplos del byte.
- Interpretar y asignar permisos de archivos y directorios.
- Explicar por qué existen los sistemas de archivos transaccionales.

---

## 1. El sistema informático y el software

### ¿Qué es un sistema informático?

Un **sistema informático** es la combinación de tres elementos que se necesitan mutuamente: **hardware** (los componentes físicos), **software** (los programas que hacen que ese hardware sirva para algo) y **usuarios** (personas u otros sistemas que lo utilizan). Ninguno de los tres funciona de forma útil por sí solo: el hardware sin software es inerte —como un coche sin conductor ni gasolina—, y el software sin hardware no tiene dónde ejecutarse.

### Software de base y software de aplicación

Dentro del software, hay una distinción fundamental:

- **Software de base**: el que permite que el resto del sistema funcione. Incluye el sistema operativo, los controladores (drivers) y ciertas utilidades del sistema (antivirus, gestores de particiones...). Existe para que las aplicaciones no tengan que "hablar" directamente con el hardware.
- **Software de aplicación**: los programas que usa el usuario para realizar tareas concretas — un procesador de textos, un navegador, un videojuego.

Esta distinción no siempre es perfectamente nítida (por ejemplo, un antivirus se percibe como una aplicación más, pero funciona a bajo nivel dando soporte al resto del sistema), pero como regla general: si un programa existe para que el sistema funcione, es software de base; si existe para que el usuario haga algo concreto, es software de aplicación.

### Niveles del sistema informático

Todo sistema informático se puede representar como una serie de capas, de más cercana al hardware a más cercana al usuario:

```
Usuario
    ↑
Software de aplicación
    ↑
Lenguajes / entornos de programación
    ↑
Software de base (sistema operativo, drivers)
    ↑
Hardware
```

Cada capa se apoya en la anterior y ofrece una base más sencilla de usar a la siguiente. El hardware es lo único que existe físicamente; todo lo demás son distintos niveles de software que hacen que ese hardware sea utilizable.

### ¿Dónde encajan los lenguajes de programación?

Entre el software de base y el software de aplicación existe otra capa: los **lenguajes de programación**, con los que se construye el software de aplicación (y también parte del software de base). Existen distintos niveles de lenguaje:

- **Lenguaje máquina**: instrucciones binarias que la CPU ejecuta directamente. Es el único lenguaje que el hardware entiende de verdad.
- **Ensamblador**: una representación algo más legible del lenguaje máquina, con una correspondencia casi directa entre instrucción y operación de la CPU.
- **Lenguajes de alto nivel** (Python, Java, C++...): mucho más cercanos a cómo razona una persona, pero que necesitan traducirse (mediante un compilador o un intérprete) antes de convertirse en instrucciones que la CPU pueda ejecutar.

No hace falta que sepas programar para esta unidad — lo importante es que entiendas que cuando alguien programa una aplicación, ese código termina bajando, nivel a nivel, hasta convertirse en instrucciones que la CPU ejecuta directamente.

### Para practicar

**Actividad:** clasifica los siguientes elementos como hardware, software de base o software de aplicación, y justifica tu respuesta en una frase: el núcleo de Linux, un navegador web, el controlador de una tarjeta gráfica, una memoria RAM, un editor de código, un antivirus, el firmware de la BIOS/UEFI.

**Ejemplo resuelto:** *el controlador de una tarjeta gráfica* → software de base, porque no es una herramienta que el usuario utilice directamente para una tarea, sino una pieza de software que permite que el sistema operativo se comunique con ese componente de hardware concreto.

Resuelve el resto por tu cuenta, siguiendo el mismo razonamiento.

---

## 2. Funciones y arquitectura del sistema operativo

### Las cuatro grandes funciones del sistema operativo

El sistema operativo cumple, de forma general, cuatro funciones:

- **Gestión de procesos**: reparte el tiempo de CPU entre los distintos programas en ejecución.
- **Gestión de memoria**: organiza la memoria RAM para que las aplicaciones no interfieran entre sí.
- **Gestión de archivos**: organiza y da acceso ordenado a la información guardada en el almacenamiento.
- **Gestión de entrada/salida**: media entre las aplicaciones y los periféricos (teclado, ratón, impresora, red...).

En esta unidad no vas a ver *cómo* decide el sistema operativo, por ejemplo, qué proceso ejecutar en cada instante (eso son los algoritmos de planificación) ni *cómo* calcula exactamente el reparto de la memoria — eso queda fuera del alcance de este curso. Lo importante aquí es que entiendas *qué* problema resuelve cada función.

### Arquitectura del sistema operativo por capas

El sistema operativo, a su vez, se organiza también por capas:

```
Aplicaciones
    ↑
Shell (interfaz de usuario)
    ↑
Controladores (drivers)
    ↑
Núcleo (kernel)
    ↑
Hardware
```

- **Núcleo (kernel)**: la capa más interna, la única que se comunica directamente con el hardware. Gestiona procesos, memoria y dispositivos a bajo nivel.
- **Controladores (drivers)**: piezas de software, normalmente suministradas por el fabricante de cada componente, que permiten al núcleo comunicarse con un dispositivo concreto.
- **Shell**: la capa que permite al usuario dar instrucciones al sistema. Puede ser en **texto** (una línea de comandos, como bash o PowerShell) o en **gráficos** (un escritorio con ventanas e iconos).
- **Aplicaciones**: los programas que usa el usuario, apoyados en todas las capas anteriores.

### ¿Y la interfaz gráfica?

Es habitual pensar que la interfaz gráfica (el escritorio, las ventanas, los iconos) *es* el sistema operativo — sobre todo si nunca has usado un ordenador de otra forma. Pero no es así: la interfaz gráfica es simplemente **la versión gráfica de la capa de shell**, no una capa nueva ni el sistema operativo en sí.

La prueba más clara es que un sistema operativo puede funcionar perfectamente **sin ninguna interfaz gráfica**: es lo habitual en los servidores, que se administran por red y no necesitan que nadie se siente delante con teclado y ratón. El sistema operativo sigue estando completo —núcleo, gestión de procesos, memoria, archivos— aunque le falte la capa de shell en su versión gráfica.

### Tipos de núcleo

No todos los núcleos se diseñan de la misma forma. Existen tres modelos:

- **Monolítico**: todos los servicios básicos (procesos, memoria, drivers...) se ejecutan juntos, en el mismo espacio privilegiado. Es rápido, pero un fallo en cualquier parte puede afectar a todo el núcleo. Es el caso de **Linux** (aunque es modular: puede cargar y descargar partes de su funcionalidad en caliente, mediante módulos).
- **Microkernel**: el núcleo se reduce al mínimo imprescindible, y el resto de servicios se ejecutan como procesos independientes y aislados entre sí. Es más robusto ante fallos, a cambio de más coste de rendimiento por la comunicación constante entre piezas separadas.
- **Híbrido**: combina ideas de ambos modelos, buscando un equilibrio. Es el caso de **Windows NT** (la base de todos los Windows modernos) y también de **macOS/iOS** (que usa un núcleo llamado XNU).

No existe una jerarquía de calidad entre estos tres modelos: son decisiones de diseño distintas, cada una con sus ventajas e inconvenientes. Que Linux use un núcleo monolítico no lo hace "anticuado" frente a un microkernel, ni un núcleo híbrido es automáticamente "lo mejor de ambos mundos" sin matices.

### Windows y Linux, capa a capa

| Elemento | Windows | Linux |
|---|---|---|
| Núcleo | Núcleo NT (híbrido) | Núcleo Linux (monolítico modular) |
| Shell en texto | CMD / PowerShell | bash / zsh |
| Shell gráfico | Explorador de Windows | GNOME / KDE, entre otros |
| Gestión de software | Instaladores .exe/.msi, Microsoft Store, winget | Gestores de paquetes (APT, DNF) |
| Controladores | Suministrados por el fabricante, vía Windows Update o instalación manual | Suministrados por el fabricante o integrados en el propio núcleo |

### Para practicar

**Actividad:** para cada una de estas situaciones, indica en qué capa de la arquitectura del sistema operativo se soluciona el problema (núcleo, controladores, shell, aplicaciones): "el ordenador no reconoce una impresora nueva", "quieres escribir un comando para renombrar 200 archivos a la vez", "una aplicación se cierra sola de forma repetida", "el sistema reparte la CPU entre varios programas abiertos a la vez".

**Ejemplo resuelto:** *"el ordenador no reconoce una impresora nueva"* → controladores: el sistema necesita el driver adecuado para poder comunicarse con ese dispositivo concreto.

Resuelve el resto por tu cuenta.

---

## 3. Procesos y sus estados

### Programa y proceso no son lo mismo

Un **programa** es código almacenado —un archivo en disco, sin ejecutar—. Un **proceso** es ese programa **en ejecución**, con recursos ya asignados por el sistema operativo (memoria, tiempo de CPU). La diferencia importa: puedes tener un único archivo de programa (por ejemplo, un navegador) y sin embargo varios procesos de ese mismo programa ejecutándose a la vez.

### Estados de un proceso

A lo largo de su vida, un proceso pasa por distintos estados:

- **Nuevo**: se acaba de crear.
- **Listo**: está preparado para ejecutarse, esperando su turno de CPU.
- **En ejecución**: la CPU está procesando sus instrucciones en ese instante.
- **Bloqueado**: está esperando algún recurso externo (datos del disco, respuesta de la red...) y no puede avanzar hasta que llegue. Es un estado normal y temporal, no un fallo.
- **Terminado**: ha finalizado su ejecución.

En esta unidad no vas a estudiar cómo decide el sistema operativo qué proceso ejecutar en cada momento —eso se conoce como planificación de procesos y queda fuera del alcance de este curso—, pero sí vas a retomar este contenido más adelante, cuando trabajes con herramientas reales como el Administrador de tareas de Windows o el comando `top` de Linux, que muestran estos procesos y estados en vivo.

### Para practicar

**Actividad:** describe, usando los estados vistos, por qué pasa un proceso de navegador cuando abres una página web: desde que abres el navegador hasta que la página termina de cargar.

**Ejemplo resuelto (primera parte):** al abrir el navegador, el proceso pasa de **nuevo** a **listo**, y en cuanto la CPU le asigna tiempo, pasa a **ejecución**.

Continúa tú la secuencia hasta que la página termina de cargar, identificando en qué momento el proceso pasa a estar **bloqueado** y por qué.

---

## 4. Sistema de archivos: organización y atributos

### ¿Para qué sirve un sistema de archivos?

Cuando guardas un archivo, no se coloca en un lugar aleatorio del disco: el **sistema de archivos** es la estructura que organiza y localiza toda la información almacenada. Sin él, un disco sería simplemente un espacio de bytes sin ningún orden ni forma de encontrar nada.

### La jerarquía de directorios en Windows

Windows organiza los discos con **letras de unidad** (`C:\`, `D:\`...), cada una con su propia estructura de carpetas. Dentro de la unidad principal encontrarás carpetas típicas como `Usuarios`, `Archivos de programa` o `Windows`. Las rutas se escriben separando carpetas con la barra invertida `\`, y pueden ser:

- **Absolutas**: desde la raíz de la unidad, por ejemplo `C:\Usuarios\Alejandro\Documentos\informe.docx`.
- **Relativas**: desde la carpeta en la que ya estás, por ejemplo `Documentos\informe.docx` si ya estás dentro de `C:\Usuarios\Alejandro`.

### La jerarquía de directorios en Linux

Linux no usa letras de unidad: existe una **única raíz** (`/`), y cualquier disco o partición adicional se **monta** dentro de ese árbol único, apareciendo como una carpeta más — no como una letra nueva. Algunos de los directorios principales son:

- `/home` — carpetas personales de cada usuario.
- `/etc` — archivos de configuración del sistema.
- `/var` — datos que cambian con el tiempo (registros, colas de impresión...).
- `/bin` y `/usr` — programas del sistema.

Las rutas en Linux se separan con la barra `/` (no `\`), y también pueden ser absolutas o relativas, igual que en Windows.

### La diferencia que más suele fallar: mayúsculas y minúsculas

**Windows no distingue mayúsculas de minúsculas** en los nombres de archivo: `Documento.txt` y `documento.txt` son el mismo archivo. **Linux sí distingue**: esos mismos dos nombres serían dos archivos completamente distintos. Esta diferencia es la causa más habitual de que "un comando no funcione" cuando se empieza a trabajar con la terminal de Linux — conviene tenerla muy presente desde ya, antes de que te encuentres con ese problema en la práctica.

### Atributos de archivos y directorios

Tanto Windows como Linux permiten marcar archivos y carpetas con **atributos** que indican propiedades adicionales:

- **En Windows**: solo lectura, oculto, sistema, archivo listo para archivar. Se consultan y modifican desde las propiedades del archivo en el Explorador.
- **En Linux**: los archivos ocultos se marcan poniendo un punto al principio del nombre (`.archivo`). Existen otros atributos más avanzados gestionables con la herramienta `chattr`, que verás con más detalle en unidades posteriores.

Un archivo oculto **no está protegido** de ningún modo especial: simplemente no se muestra por defecto. No confundas "oculto" con "seguro".

### Para practicar

**Actividad (rutas):** dado el siguiente fragmento de árbol de directorios Linux, escribe la ruta absoluta y una ruta relativa posible (suponiendo que partes de `/home/alumno`) hasta el archivo `informe.txt`:

```
/
├── home/
│   └── alumno/
│       └── documentos/
│           └── informe.txt
├── etc/
└── var/
```

**Ejemplo resuelto:** ruta absoluta → `/home/alumno/documentos/informe.txt`. Ruta relativa desde `/home/alumno` → `documentos/informe.txt`.

**Actividad (mayúsculas):** de estos tres nombres de archivo — `Informe.docx`, `informe.docx`, `INFORME.docx` —, ¿cuántos archivos distintos son en Windows? ¿Y en Linux? Justifica tu respuesta con lo aprendido en este apartado.

---

## 5. Binario y octal aplicados a permisos, codificación de caracteres y unidades de medida de la información

### Por qué el ordenador usa binario

Un transistor solo tiene dos estados posibles: encendido o apagado. Por eso los ordenadores representan toda la información en **binario** (base 2, con solo dos dígitos: 0 y 1) — no es una elección arbitraria, es una consecuencia directa de cómo funciona el hardware.

### Conversión entre decimal y binario

Cualquier número decimal se puede representar en binario. Por ejemplo, el número decimal 13 se descompone como potencias de 2:

$$13 = 8 + 4 + 1 = 2^3 + 2^2 + 2^0 \Rightarrow 1101 \text{ en binario}$$

Y al revés, para convertir un binario a decimal, se suman las potencias de 2 correspondientes a cada posición con un 1: `1101` → 1×8 + 1×4 + 0×2 + 1×1 = 13.

En esta unidad trabajarás con números de hasta un byte (8 bits, valores de 0 a 255) — lo suficiente para lo que necesitas: leer y escribir permisos.

### Codificación de caracteres: de binario a texto

Un byte no solo sirve para representar cantidades: también puede representar un **carácter** de texto, según qué tabla de correspondencia se use.

- **ASCII** fue el primer sistema de codificación ampliamente usado: asigna un valor numérico (de 0 a 127) a cada letra del alfabeto inglés, los dígitos y algunos símbolos. Es suficiente para escribir en inglés, pero **no incluye acentos, eñes, ni alfabetos distintos del latino** — se quedó corto en cuanto la informática se extendió más allá del mundo anglosajón.
- **Unicode** resuelve esa limitación asignando un identificador único a prácticamente cualquier carácter de cualquier idioma o sistema de escritura existente (y también a símbolos como los emojis). Unicode se diseñó para ser compatible con ASCII: sus primeros 128 caracteres coinciden exactamente.
- **UTF-8** y **UTF-16** son formas distintas de convertir esos identificadores Unicode en bytes concretos. **UTF-8** es, con diferencia, la más usada hoy en la web y en Linux, precisamente porque mantiene esa compatibilidad con ASCII a nivel de bytes.

Cuando un archivo de texto se abre "lleno de símbolos raros" en vez de mostrar tildes o eñes correctamente, casi siempre es un problema de codificación: los bytes son los mismos, pero se están interpretando con la tabla de correspondencia equivocada.

### Sistema octal: por qué nos interesa aquí

El sistema **octal** (base 8) agrupa los bits de tres en tres para representarlos con un solo dígito (0-7). No es casualidad que se use precisamente para permisos: **un permiso `rwx` son exactamente 3 bits** (uno por cada tipo de permiso, activado o desactivado), así que **un dígito octal representa un conjunto completo de permisos**. Es la razón real de que se use octal aquí, y no otra base — por ejemplo, hexadecimal agrupa de 4 en 4 bits y no encajaría de forma tan limpia con un conjunto de 3 permisos.

### Unidades de medida de la información

El byte es la unidad base para medir cantidades de información, pero sus múltiplos se definen de **dos formas distintas**, según dos normas diferentes:

- El **Sistema Internacional (SI)**: kilo, mega, giga... en **base 10** (potencias de 1000). Es el que usan los fabricantes de discos para anunciar su capacidad: 1 kB = 1000 bytes, 1 MB = 1000 kB, 1 GB = 1000 MB.
- La norma **IEC 80000-13**: kibi, mebi, gibi... en **base 2** (potencias de 1024). Es la que suelen calcular internamente los sistemas operativos: 1 KiB = 1024 bytes, 1 MiB = 1024 KiB, 1 GiB = 1024 MiB.

El problema es que, por costumbre histórica, la mayoría de sistemas operativos calculan en GiB pero lo etiquetan en pantalla como "GB" — lo cual genera una confusión muy conocida.

#### El caso real: el disco de 500 GB que se queda en ~465 GB

Un disco anunciado como "de 500 GB" tiene exactamente 500 × 10⁹ = 500.000.000.000 bytes — ese es el Sistema Internacional, tal y como lo usa el fabricante, y no hay ningún engaño en ello. Cuando el sistema operativo muestra la capacidad de ese mismo disco, la calcula en GiB (base 2), aunque la etiquete como "GB":

$$500.000.000.000 \div 1024^3 = 500.000.000.000 \div 1.073.741.824 ≈ 465,7 \text{ GiB}$$

No se ha perdido ningún byte: son exactamente los mismos, contados con dos reglas de agrupación distintas. Es importante que entiendas este cálculo, porque en cuanto empieces a particionar discos para tus máquinas virtuales vas a comprobarlo con tus propios ojos.

### Permisos de archivos y directorios

En Linux, cada archivo o directorio tiene permisos definidos para **tres conjuntos de usuarios**: el **propietario**, el **grupo** al que pertenece, y **otros** (el resto de usuarios del sistema). Para cada conjunto existen tres tipos de permiso:

- **r** (read / lectura): poder ver el contenido.
- **w** (write / escritura): poder modificar el contenido.
- **x** (execute / ejecución): poder ejecutar el archivo si es un programa o script; en un directorio, significa poder **entrar** en él y acceder a su contenido (no "ejecutarlo").

Estos permisos se representan de dos formas equivalentes:

- **Simbólica**: por ejemplo `rwxr-xr--` — el propietario tiene lectura/escritura/ejecución, el grupo tiene lectura/ejecución, y el resto solo lectura.
- **Numérica (octal)**: la misma información, un dígito por conjunto de usuarios. `rwxr-xr--` equivale a **754**.

En **Windows**, el sistema de archivos NTFS también gestiona permisos, pero con un modelo distinto y más granular (listas de control de acceso, o ACL), que no se resume con esta misma notación numérica. Lo verás en detalle en una unidad posterior.

#### De binario a permiso, paso a paso

Para pasar de un valor octal a lo que puede hacer cada usuario, conviene descomponerlo en binario. Por ejemplo, el permiso **640**:

- 6 → `110` → lectura y escritura, sin ejecución (propietario).
- 4 → `100` → solo lectura (grupo).
- 0 → `000` → ningún permiso (resto de usuarios).

Es decir, `640` equivale a `rw-r-----`.

### Para practicar

**Actividad (codificación de caracteres):** a partir de la tabla ASCII reducida que se te proporcione, traduce una palabra corta en inglés a su secuencia de valores ASCII (uno por letra), y a la inversa, decodifica una secuencia de valores que se te dé. Después, responde: ¿por qué la palabra "Ñandú" no se puede representar completa usando solo ASCII, y sí usando Unicode?

**Actividad (unidades de medida):** calcula, mostrando el cálculo, cuántos GiB mostrará aproximadamente el sistema operativo para un disco anunciado como de 256 GB y para uno de 1 TB.

**Ejemplo resuelto (256 GB):** 256 × 10⁹ bytes ÷ 1024³ = 256.000.000.000 ÷ 1.073.741.824 ≈ 238,4 GiB.

Calcula tú el caso del disco de 1 TB.

**Actividad (binario/octal):** convierte a binario los siguientes valores decimales: 5, 12, 27, 64, 200. Después, agrupa cada resultado de 3 en 3 bits (desde la derecha) para obtener su equivalente en octal.

**Actividad (permisos):** dados estos permisos en notación simbólica — `rwxr-xr-x`, `rw-rw-r--`, `rwx------` —, obtén su notación octal. Y a la inversa, dados los valores octales 644 y 600, obtén la notación simbólica y explica qué puede hacer cada tipo de usuario.

**Ejemplo resuelto:** `rwxr-xr-x` → 755.

Resuelve el resto por tu cuenta, aplicando el mismo método (no memorices resultados: descompón cada conjunto en sus tres bits).

---

## 6. Sistemas transaccionales

### El problema que resuelven

Imagina que el ordenador se queda sin electricidad justo mientras se está escribiendo un archivo grande en el disco. Sin ningún mecanismo de protección, el sistema de archivos podría quedar en un estado inconsistente —corrupto—, con información a medio escribir y sin forma clara de saber qué se llegó a guardar y qué no.

### Journaling: cómo se resuelve

Los sistemas de archivos modernos, como **ext4** (habitual en Linux) y **NTFS** (usado por Windows), incluyen un mecanismo llamado **journaling** (o "sistema transaccional"): antes de realizar una operación de escritura, el sistema la registra en un diario (*journal*). Si la operación se interrumpe, al reiniciar el sistema puede consultar ese diario y completar o deshacer la operación de forma limpia, en vez de dejar el sistema de archivos en un estado inconsistente.

Es una de las razones por las que los sistemas operativos actuales rara vez "se rompen" tras un corte de luz o un apagado brusco, a diferencia de sistemas de archivos más antiguos sin esta protección (como FAT32).

**Importante:** el journaling protege la **consistencia del sistema de archivos**, no evita perder el contenido concreto que se estuviera escribiendo justo en el momento del corte, y **no sustituye a hacer copias de seguridad**. Son dos protecciones distintas que hacen falta las dos.

### Para practicar

**Actividad:** explica con tus propias palabras la diferencia entre que un sistema de archivos tenga journaling y que se hagan copias de seguridad periódicas, y por qué hacen falta las dos cosas.

---

## Caso práctico integrador

Un compañero de prácticas en la empresa te cuenta que ha copiado una carpeta de un proyecto desde un Linux a un pendrive formateado en Windows, y al volver a copiarla a un Linux distinto, los permisos "ya no eran los mismos": algunos scripts que antes eran ejecutables ya no lo son.

Con lo aprendido en esta unidad:

1. Explica qué sistema de archivos maneja permisos Unix de forma nativa y cuál no, y por qué eso puede explicar la pérdida de permisos al pasar por un pendrive.
2. Explica, en términos de binario/octal, qué significa que un script "deje de ser ejecutable" (qué bit ha cambiado).
3. Relaciona el caso con la arquitectura por capas: ¿en qué capa se decide qué permisos tiene un archivo?

---

## Resumen de la unidad

- Un sistema informático es hardware + software + usuarios; el software se divide en software de base y software de aplicación.
- El sistema operativo se organiza por capas (núcleo, controladores, shell, aplicaciones) y cumple cuatro funciones: gestión de procesos, memoria, archivos y entrada/salida.
- La interfaz gráfica es la versión gráfica del shell, no una capa aparte ni el sistema operativo en sí.
- Los núcleos pueden ser monolíticos (Linux), híbridos (Windows, macOS) o microkernel — son diseños distintos, no una jerarquía de calidad.
- Un proceso es un programa en ejecución, con estados: nuevo, listo, ejecución, bloqueado, terminado.
- Windows organiza los archivos con letras de unidad; Linux usa una única raíz `/` y distingue mayúsculas de minúsculas.
- El texto también se representa en binario mediante una tabla de codificación: ASCII (limitado al alfabeto inglés) y Unicode/UTF-8 (compatible con ASCII, pero capaz de representar cualquier idioma).
- El byte tiene múltiplos según dos normas: SI (base 10, el que anuncia el fabricante) e IEC 80000-13 (base 2, el que suele calcular y mal-etiquetar el sistema operativo) — de ahí que un disco de 500 GB se muestre como ~465 GB.
- Un dígito octal representa un conjunto completo de permisos `rwx` (3 bits). Los permisos se definen para propietario, grupo y otros.
- El journaling protege la consistencia del sistema de archivos ante interrupciones, pero no sustituye a las copias de seguridad.

## Relación con RA y CE

| CE | Contenido | Dónde se trabaja |
|---|---|---|
| a) | Elementos funcionales de un sistema informático | Apartado 1 |
| b) | Representación de la información | Apartado 5 |
| c) | Funciones del sistema operativo | Apartado 2 |
| d) | Arquitectura del sistema operativo | Apartado 2 |
| e) | Procesos y sus estados | Apartado 3 |
| f) | Estructura y organización del sistema de archivos | Apartado 4 |
| g) | Atributos de archivo y directorio | Apartado 4 |
| h) | Permisos de archivos y directorios | Apartado 5 |
| i) | Sistemas transaccionales | Apartado 6 |
