# UD02 — Práctica 4, anexo: solución al fallo de la instalación de Ubuntu

**Módulo:** Sistemas Operativos Monopuesto · 1º SMR

> **Para qué sirve este anexo.** En la Práctica 4, al instalar Ubuntu junto a Windows, el instalador puede detenerse y mostrar la ventana **«Send problem report to the developers?»** con el título **«partitioning crashed with CurtinInstallError»**. Si vuelves a intentarlo tal cual, fallará otra vez. Aquí tienes por qué ocurre y cómo se soluciona en cinco minutos.

## El síntoma

- La instalación se detiene durante el particionado, con un aspa roja en la ventana del instalador.
- Aparece un informe de error de *subiquity*, el instalador de Ubuntu, con el título **`partitioning crashed with CurtinInstallError`**.
- Entre los datos del informe puede aparecer `BootstrapLog: Error: [Errno 40] Too many levels of symbolic links`. **Ignóralo:** es un fallo del propio informe, no la causa del problema.

## Por qué ocurre

El disco de la VM usa una tabla de particiones **GPT**. GPT guarda la tabla **dos veces**: una al principio del disco y una copia de seguridad **al final**. Así, si una se estropea, se puede recuperar con la otra.

En el apartado 1 de la práctica ampliaste el disco virtual de 64 GB a 100 GB desde VirtualBox. VirtualBox añadió el espacio, pero **no movió la copia de la tabla**, que se quedó donde antes acababa el disco:

```
Antes de ampliar (64 GB):
[GPT principal][ EFI | MSR | C: (Windows) | Recuperación ][GPT copia]

Después de ampliar (100 GB):
[GPT principal][ EFI | MSR | C: (Windows) | Recuperación ][GPT copia][ 36 GB libres ...... ]
                                                              ↑
                                         la copia ya no está al final del disco
```

Windows funciona igual, pero el instalador de Ubuntu encuentra una tabla de particiones incoherente: la copia no está donde debería y estorba en mitad del espacio libre. En lugar de arriesgarse a estropear el disco, se detiene.

**La solución:** mover la copia de la tabla al nuevo final del disco. Lo hace la herramienta `parted` con una sola respuesta, y **no toca ninguna partición ni ningún dato**.

## Solución paso a paso

**1. Cierra el informe de error.** Pulsa **Don't send**: enviarlo no soluciona nada. Cierra también la ventana del instalador si sigue abierta. Seguirás en la sesión en vivo de Ubuntu, con su escritorio.

**2. Abre una terminal.** Pulsa **Ctrl+Alt+T**, o busca «Terminal» en el menú de aplicaciones.

**3. Revisa las tablas de particiones con `parted`:**

```bash
sudo parted -l
```

`parted -l` muestra los discos y sus particiones. Al leer el disco de la VM detectará el problema y te preguntará qué hacer. El mensaje será parecido a uno de estos (pueden salir los dos, uno detrás de otro):

```
Warning: Not all of the space available to /dev/sda appears to be used,
you can fix the GPT to use all of the space (an extra 75497472 blocks) or
continue with the current setting?
Fix/Ignore?
```

```
Error: The backup GPT table is not at the end of the disk, as it should be.
Fix, by moving the backup to the end (and removing the old backup)?
Fix/Cancel?
```

Si la sesión en vivo está en español, verás lo mismo traducido («Arreglar/Ignorar»).

**4. Responde `Fix`** (escribe `Fix` o `F` y pulsa Intro) a cada pregunta. Si te sale en español, la opción es `Arreglar`.

> **Ojo:** responde `Fix` **solo** a estos mensajes sobre el espacio sin usar o la copia de la GPT del disco de la VM (`/dev/sda`). Si `parted` te preguntara cualquier otra cosa, responde `Ignore` o `Cancel` y consúltalo con el profesor.

**5. Comprueba que está arreglado.** Vuelve a ejecutar:

```bash
sudo parted -l
```

Esta vez no debe aparecer ninguna pregunta. El disco `/dev/sda` debe medir unos **107 GB** (son los 100 GiB de VirtualBox expresados en GB) y deben verse las particiones de Windows.

**6. Reinicia la VM** con la ISO de Ubuntu todavía montada y vuelve a empezar la instalación desde el apartado 2 de la práctica.

**7. Antes de crear la partición de Ubuntu, revisa la tabla.** En la instalación manual solo deben aparecer las cuatro particiones de Windows (EFI, reservada de Microsoft, C: y recuperación) y el **espacio libre**. Si quedó alguna partición ext4 de un intento anterior, que sea **tuya**, bórrala para que vuelva a ser espacio libre. **No toques las de Windows.**

## Si no has llegado todavía a instalar Ubuntu

Puedes evitar el fallo antes de que ocurra. Arranca la VM con la ISO de Ubuntu y, en la primera pantalla del instalador, elige **probar Ubuntu** en lugar de instalar (o cierra el instalador). Abre una terminal y haz los pasos 3 a 5. Después lanza el instalador desde el icono del escritorio.

## Si el problema no es este

Si `sudo parted -l` **no** te pregunta nada, la causa es otra. Busca el error real en el registro del instalador:

```bash
sudo grep -iE "error|fail" /var/log/installer/curtin-install.log | tail -n 25
```

Las otras dos causas habituales en esta práctica son:
- **La partición de Windows no se cerró bien** (cifrado activo o hibernación): repasa los pasos 3 y 4 del apartado 1.
- **Un error en el particionado manual**: formatear la partición EFI de Windows, crear una EFI nueva o tocar una partición de Windows.

Si tenías la instantánea **«Antes de Ubuntu»**, restaurarla es la forma más limpia de repetir la instalación.

## Para tu entrega

Esto es una **incidencia de instalación** de las que pide el **apartado 5** de la práctica. Si te ha ocurrido, documéntala con:
- **síntoma:** captura del informe de error;
- **diagnóstico:** el comando que usaste y captura del aviso de `parted`;
- **causa:** con tus palabras;
- **solución:** qué respondiste;
- **comprobación final:** `parted -l` sin avisos y la instalación terminada.

Un técnico no recuerda todas las soluciones: sabe dónde mirar.
