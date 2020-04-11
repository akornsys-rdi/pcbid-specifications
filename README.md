# Especificaciones de PCBID V2.0 - Borrador

![Akornsys RDI](https://github.com/akornsys-rdi/pcbid-specifications/raw/master/doc/img/akornsys-logo.png)

## :rocket: Descripción e intención

PCBID es un sistema de numeración única para PCB diseñado para generar **códigos únicos** que identifican las PCB y permiten su **trazabilidad**. Esta numeración ha sido concebida para ser un sistema robusto, **usable ante cualquier escenario**. Además **aporta información útil** para el usuario, asegurando una **buena legibilidad** de la numeración.

El sistema se ha ideado para aportar una **alta compresión de datos**, creando un **código suficientemente corto** como para ser insertado incluso en las PCB más pequeñas. Y el proceso de generación, así como el de sobreimpresión permiten ser **procesos automatizados** con herramientas externas. De esta menera, se permiten en la versión actual más de 50 millones de autores, con un total de más 2700 billones de proyectos únicos.

### Tamaño recomendado en serigrafía

Del mismo modo que no hay una única forma de hacer una PCB, no hay un tamaño único que se considere bueno para la impresión del PCBID en una PCB. Sin embargo, sí consideramos que para mantener una buena legibilidad, no debería ser menor de 30x6mil.

### Character set

PCBID usa codificación en K85, una variante de ASCII85 que reemplaza algunos caracteres similares por otros, para evitar confusiones tipográficas.

K85 consta del siguiente charset:

        0 1 2 3 4 5 6 7 8 9 A B C D E F G H J K L M N P Q R S T U V W X Y Z
        a b c d e f g h i j k l m n p q r s t u v w x y z ! # $ % & ( ) * +
        - ; < = > ? @ ^ _ { } [ ] / \ : "

Este charset ha sido considerado con cuidado. De esta forma queda excluído:
- `"I"` y `"|"`, capaz de confundirse con `"1"`, `"i"` y `"l"`.
- `"O"` y `"o"`, capaz de confundirse con `"0"` y `"Q"`.
- ``"`"`` y `"'"` que son demasiado pequeñas y pueden no ser legibles en serigrafía.
- `" "` por ser caracter no imprimible.
- `"~"` que puede ser usado como indicador de PCBID al usarse delante de éste.

### Descripción de campos

El código de PCBID consta de seis campos:
- Autor: Identificador de la persona u organizacion que desarrolla la PCB.
- Proyecto: Identificador del proyecto desarrollado por el autor.
- Módulo: Numeración del número de placa que pertenece al proyecto. El campo empieza en 01.
- Versión: Numeración de la versión del presente módulo. El campo empieza en 01.
- Semana: Semana de diseño de la PCB.
- Año: Año de diseño de la PCB.

La estructura canónica de los campos está diseñada en forma de dependecia, por lo que el campo a la derecha y sucesivos son dependientes del campo anterior. De esta forma los campos pueden leerse en el orden opuesto: _Fecha de diseño_ DE _la versión_ DE _el módulo_ DE _el proyecto_ DE _el autor_. Un cambio en un campo obliga al reinicio de los campos sucesivos.

De esta forma PCBID queda así constituído:

        ~aaaappppmmrrwwyy       ~64M]";Br01011520
         aaaa                    64M]               Autor       Foobar Inc
             pppp                    ";Br           Proyecto    Foobar Project
                 mm                      01         Módulo      1
                   rr                      01       Versión     1
                     ww                      15     Semana      15
                       yy                      20   Año         2020

### 2D Barcode

La numeración de PCBID también puede ser codificada en formato de código de barras para ser añadida en la serigrafía de la PCB. El formato de código de barras para la codificación del PCBID es **datamatrix**. Es altamente recomendable añadir el código PCBID en texto en una ubicación próxima al código de barras siempre que sea posible. Da buen resultado generarlo en colores invertidos y con un tamaño en PCB no inferior a 6mm, siempre y cuando el color de la máscara ofrezca suficiente contraste.

### ¿Cuándo debo generarlo?

Con el fin de que la fecha embebida en el código PCBID sea válida, ha de ser generado e insertado una vez finalizada la PCB, en el momento previo a generar los archivos de fabricación. Este código tendrá validez y no deberá ser actualizado hasta el siguiente cambio en la PCB.

### Ejemplos de uso

#### Caso 1: Primera PCB

1. Genera tu identificador de autor a partir del nombre de autor que desees (nombre personal o de marca). Ej. `64M]` para `Foobar Inc`.
2. Genera tu identificador de proyecto a partir del nombre del proyecto. Ej. `";Br` para `Foobar Project`.
3. Establece los campos de módulo y versión a `01`.
4. Calcula la semana actual siguiendo el modelo ISO 8601 (`%V` en sintaxis `date`). Ej. `15` para `11/04/20`.
5. Añade las últimas dos cifras del año para el campo año. Ej: `20` para `2020`.
6. Concatena los campos, de esta manera ha de quedar: `64M]";Br01011520`.
7. Inserta el PCBID en la PCB. Puedes insertar `~` al inicio para indicar que se trata de un PCBID. `~64M]";Br01011520`.

#### Caso 2: Segunda versión de la PCB

1. Coge el PCBID que tenía asignado la versión anterior de esta PCB. Ej: `~64M]";Br01011520`.
2. Incrementa en una unidad, siguiendo el esquema de caracteres K85, el campo versión. Ej: `~64M]";Br01021520`.
3. Recalcula la fecha cuando hayas finalizado los cambios de la segunda versión de la PCB. Ej: `~64M]";Br01021820`.
4. Inserta el PCBID en la PCB.

#### Caso 3: Nueva placa en el mismo proyecto

1. Coge los campos autor y proyecto que tenías generados de este proyecto. Ej: `64M]";Br`.
2. Incrementa en una unidad, siguiendo el esquema de caracteres K85, el campo módulo. Ej: `02`.
3. Reinicia el campo versión, dado que es la primera versión de esta PCB. Ej: `01`.
4. Cuando hayas acabado el diseño, calcula los campos de fecha. Ej: `2120`.
5. Concatena los campos, de esta manera ha de quedar: `~64M]";Br02012120`.
6. Inserta el PCBID en la PCB. De esta manera tu proyecto con identificador `";Br` consta de la presente PCB con PCBID `~64M]";Br02012120` y de otra PCB en su segunda versión con PCBID `~64M]";Br01021820`.

### FAQ

> **Hace un tiempo diseñé y fabriqué unas PCB con PCBID, ahora quiero fabricar más PCB a partir de los mismos archivos de fabricación. ¿Debo actualizar el PCBID?**
> 
> En el uso habitual de PCBID no es necesario actualizar la numeración. No obstante, si es un evento que necesitas registrar para la trazabilidad, es legítimo el incremento del campo versión en una unidad cada vez que se vayan a fabricar nuevas PCB.

> **He diseñado un proyecto de varias placas y todas ellas tienen PCBID. He creado una segunda versión de una de ellas, ¿tengo que actualizar el PCBID en todas?**
> 
> No, sólo necesitas actualizar el campo versión incrementando una unidad en la placa de la que has creado una segunda versión.

> **¿Debo actualizar el PCBID si había diseñado una PCB y después de un tiempo realizo cambios, sin haber fabricado ninguna unidad de esta PCB?**
> 
> No. Realiza todos los cambios necesarios y mantén el PCBID que tuvieses generado.

> **¿Puedo usar este sistema para la carcasa o hardware que no sean PCB de mi proyecto?**
> 
> No es la intención de PCBID y probablemente no sea la opción más conveniente. Los procesos de fabricación de PCB distan mucho de otros procesos de fabricación de otro hardware, y es posible que sea necesario realizar trazabilidades de este hardware de forma diferente que con las PCB. En cualquier caso eres libre de usarlo si así lo deseas.

> **¿Cómo puede tener trazabilidad si todas las PCB fabricadas a partir de los mismos archivos de fabricación tienen el mismo PCBID?**
> 
> PCBID se sustenta en la idea de que todas las placas fabricadas a partir de los mismos archivos de fabricación son virtualmente idénticas, por lo que todos los eventos de trazabilidad les afectan por igual. Por tanto se enfoca más en los cambios que sufre una PCB desde la versión early proto hasta la final que en aquellos eventos individuales. No obstante, sabemos que hay situaciones en las que es necesario llevar control de trazabilidad de forma individual, para ese caso recomendamos añadir otro sistema de numeración de forma paralela, como pueden ser pegatinas numeradas.

> **¿Cómo evita el sistema posibles colisiones de identificador de autor o proyecto?**
> 
> La comprobación de colisiones se realiza actualmente de forma manual, aunque está prevista su automatización con herramientas externas y con la implementación de PCBDB. No obstante, no es algo preocupante dado que la colisión de estos campos es poco probable en esta fase de desarrollo.

> **¿Qué es PCBDB?**
> 
> Es un proyecto relacionado con PCBID que se desarrollará en un futuro y permitirá ser base de datos de los eventos de trazabilidad de aquellas placas registradas que usen PCBID. Además, permitirá enlazar especificaciones y documentación útil de dichas placas.

> **¿Puedo usar este sistema para mis proyectos de software?**
> 
> No es la intención de PCBID y probablemente no sea la opción más conveniente. Los sistemas de control de versión de software aportan sus propias ventajas e incluyen sus propios sistemas de trazabilidad. En cualquier caso eres libre de usarlo si así lo deseas.

> **Mi proyecto ha cambiado de nombre, ¿debo de cambiar el campo de proyecto del PCBID?**
> 
> Si aún no has fabricado ninguna PCB de este proyecto con el nombre antiguo, genera el PCBID con el nombre nuevo de proyecto. Sin embargo, si ya has fabricado PCB del proyecto con el nombre antiguo y tiene asignado un PCBID, deberías mantener el identificador de proyecto del PCBID que ya tienes generado para las placas con el nombre de proyecto nuevo, por razones de trazabilidad. Si la razón del cambio de nombre del proyecto es por un cambio estructural del proyecto, quizá sí deberías tratarlo como un proyecto nuevo y generar su propio identificador de proyecto. En última instancia queda a tu discrección.

> **¿Por qué este sistema de numeración y no cualquier otro?**
> 
> En el mundo de fabricación de PCB hay numerosos sistemas de numeración con la intención de tener trazabilidad, pero ninguno de ellos es estándar y su ámbito suele ser uso interno de cada empresa u organización. Por ello se ha diseñado PCBID, para crear unas especificaciones estándar que pueda ser adoptado por todo el que pueda estar interesado. Además, esto permite crear una base de datos de todas aquellas que usen un mismo sistema (PCBDB). Por supuesto eres libre de usar el sistema que más de convenzca.

> **Estoy diseñando un ecosistema de proyectos que consta de varias PCB. ¿Debería cambiar el campo de proyecto o el de módulo para cada una de ellas?**
> 
> Depende. Es una cuestión semántica, pero si se puede entender que cada placa es un proyecto en sí mismo y no requiere otras de su ecosistema para funcionar probablemente debas cambiar el campo de proyecto. Por ejemplo diferentes tarjetas de un sintetizador de rack, la agrupación de ellas forman una unidad pero cada una es autónoma por separado. Si por el contrario hay cierta dependecia entre las diferentes placas para poder funcionar probablemente debar cambiar el campo de módulo. Por ejemplo una placa principal con un conjunto de placas de expansión. En última instancia queda a tu discrección.

> **Voy a fabricar una copia de una PCB diseñada por otra persona y que contiene PCBID, ¿debo mantenerlo?**
> 
> En general es una buena práctica conservarlo, pues de esta manera se puede acceder al historial de trazabilidad del autor original y se reconoce la autoría del proyecto, que siempre está bien. En caso de que no desees esto, puedes eliminarlo o regenerarlo con tu propio identificador de autor, mantener el campo de proyecto y reiniciar el campo de versión.

> **He diseñado una PCB muy pequeña y no me entra el PCBID, ¿puedo acortarlo?**
> 
> No. Todos los campos que forman PCBID tienen relevancia y no se pueden descartar algunos. Puedes reducir el tamaño de fuente al mínimo recomendado o si ya lo has hecho puedes considerar otras opciones, como insertarlo en dos líneas, o quizá insertarlo en formato datamatrix. Si eliges ponerlo en dos líneas, has de hacerlo preferentemente con el siguiente formato:
> 
>       aaaapppp            64M]";Br
>       mmrrwwyy            01011520

## :book: Version history (PCBID V1.0)

The previous version of PCBID is considered obsolete, and is retained only as documentation. This version was designed to be used by a single author as a way to generate serial numbers for personal projects

        SN:k7759C62C01a_4618
        SN:                     V1 mark
           k                    Author
            7759C62C            CRC32 of the project URI in HEX
                    01          PCB module number
                      a         PCB release
                       _        Spacer
                        46      Week
                          18    Year

## :heart: Community and contributions

[![Donate using Liberapay](https://github.com/akornsys-rdi/pcbid-specifications/raw/master/doc/img/liberapay-donate.png)](https://liberapay.com/RileyStarlight/donate)

Contributions are always welcome!

This is a **community-driven open source project**. Any contribution is highly appreciated, whether you are helping [fixing bugs or proposing new features](https://github.com/akornsys-rdi/pcbid-specifications/issues "Issues"), [pulling requests](https://github.com/akornsys-rdi/pcbid-specifications/pulls "Pull Requests"), improving the documentation or spreading the word.

Please :star: or watch this repository if this project helped you! You can also support this project on [Liberapay](https://liberapay.com/RileyStarlight/donate).

### ¿How to Pull Request to add new PCBIDs?

If you use PCBID for your own projects, please fork this repository, add your author and project data to the `authors.txt` and `projects.txt` files respectively following the instructions present in these files and make a Pull Request with the changes.

## :scroll: Copyright & License

Copyright © 2020 RileyStarlight

### Documentation

[![CC BY SA 4.0](https://github.com/akornsys-rdi/pcbid-specifications/raw/master/doc/img/ccbysa-logo.png)](https://creativecommons.org/licenses/by-sa/4.0/ "Creative Commons BY-SA 4.0")

The documentation of this project is licensed under CC BY SA 4, see the [`LICENSE.BYSA4.txt`](LICENSE.BYSA4.txt) file for details.
