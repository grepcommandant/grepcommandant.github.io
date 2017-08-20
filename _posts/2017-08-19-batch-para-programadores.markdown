---
layout: post
title:  "Batch para Programadores"
date:   2017-08-19 04:00:00 -0300
categories: notes
---

Sin dar muchas vueltas en el asunto, sabemos que este lenguaje ya es viejo. Sin embargo, muchas comunidades de desarrolladores utilizan esta herramienta para automatizar la generación de archivos de configuración que serán utilizados en un proceso posterior de un build system.

Este artículo tiene por intención resumir las sentencias, expresiones y trucos que suelen encontrarse dentro de los *.bat en las source tree de muchas herramientas que utilizamos los programadores y administradores de sistemas. Queda de más decir, esto debería servir para que puedas construir tus propios scripts.

## Comentarios

Originalmente los comentarios son escritos de la siguiente manera:

```batch
rem LINEA UNO DE COMENTARIO
rem LINEA DOS DE COMENTARIO
```
etc...

Pero gracias a un truco documentado por Marc Stern en su artículo "MS-DOS batch files Tips & Tricks", un comentario se puede escribir, y es más fácil de identificar, de la siguiente manera:

```batch
:: LINEA UNO DE COMENTARIO
:: LINEA DOS DE COMENTARIO
::LINEA TRES DE COMENTARIO
```
etc...

* NOTA: esto es truco ya que para el intérprete batch ´:´ al principio de una línea es un label, y no un comentario. Sin embargo ´::´ es un label inválido (sin nombre, vacío) y es por eso que puede utilizarse como una alternativa para líneas de comentario.

## Variables

En batch, todas las variables son globales por defecto (variables de entorno), esto es, son visibles y modificables por otros scripts. Para utilizar una copia local de las variables del entorno/session y hacer que las variables creadas con SET sean visibles localmente utilizar `setlocal` al principio del script.

```batch
setlocal
set varaible1=valor1
set varaible2=valor2
set varaible3=texto con espacios
:: Variable basada en otra
set varaible4=otro %variable3%
```

* NOTA: Por defecto, el intérprete reemplaza las referencias a las variables envueltas con % con su correspondiente valor en tiempo de parseo y no en tiempo de ejecución como es de esperar que suceda en muchos lenguajes de programación. Para cambiar este comportamiento es necesario habilitar la característica [Delayed Expansion](https://ss64.com/nt/delayedexpansion.html) y referenciar las variables de la forma `!variable!` y no `%variable%` cuando se necesite el valor de la variable en tiempo de ejecución.

* NOTA: `setlocal` se puede utilizar muchas veces en un mismo script para almacenar diferentes valores en una misma variable [SS64: SETLOCAL](https://ss64.com/nt/setlocal.html).

* NOTA: el texto o string no necesita estar entre comillas dobles. Las comillas dobles, si se encuentran envolviendo el texto, serán incluidas.

Si un valor tiene caracteres especiales entonces la expresión debe envolverse con comillas dobles (la variable NO almacenará las comillas dobles):

```batch
set "variable=one & two"
```

Si las comillas dobles se utilizan para envolver el valor, y no la expresión, entonces la variable SI almacenará las comillas dobles:

```batch
set variable="one & two"
```

* NOTA: El nombre de una variable puede contener espacios. Así, `set myvar =mytext` creará una variable llamada `myvar ` (notar que hay un espacio adelante).

* NOTA: Hay que tener cuidado con el uso exhaustivo de variables en entornos Windows 95/98 ya que utilizar demasiado espacio puede llevar a un problema conocido como ["Out of environment space"](http://www.pcguide.com/ts/x/sys/operrEnvironmentSpace-c.html)

Para obtener el valor de una variable, envolver el nombre de la misma con %:

```batch
echo %variable%
echo %myar %
```

Para ejecutar un comando cuyo filename se encuentra almacenado en una variable, simplemente usar dicha variable en lugar del comando. Ejemplo:

```batch
%MSBUILD% "project.proj"
```

Para más información ver:
 - [Windows Environment Variables - SS64](https://ss64.com/nt/syntax-variables.html).
 - [Macros - SS64](https://ss64.com/nt/syntax-macros.html).
 - [Delayed Expansion - SS64](https://ss64.com/nt/delayedexpansion.html).
 - [SET & SET /A (Environment variable arithmetic) - SS64](https://ss64.com/nt/set.html).
 - [Substring - SS64](https://ss64.com/nt/syntax-substring.html).
 - [Search & replace part of a variable - SS64](https://ss64.com/nt/syntax-replace.html).

## Variables built-in o standard

Ver [Windows Environment Variables - SS64](https://ss64.com/nt/syntax-variables.html)

## Substring de Variables

Es posible obtener caracteres específicos de las variables que contienen un string mediante la sintaxis `%variable:~num_chars_to_skip%` o `%variable:~num_chars_to_skip,num_chars_to_keep%`. Así, por ejemplo, si `%opt%` devuelve `--without-extensions`, entonces `%opt:~0,10%` devuelve `--without-`.

## Variables y Paths

 - Muchas veces es necesario trabajar dentro del directorio en donde se encuentra el script. Para obtener el path de este directorio usar `%~dp0`.
 - `%CD%` nos devuelve el directorio actual desde donde ejecutamos el script. `%CD%` no necesariamente es el mismo path que `%~dp0`.
 - Las expresiones con extensiones de argumento como `%~dp0` nos devuelve un path CON un \ al final, pero las variables built-in como `%CD%` nos devuelve un path SIN un \ al final. Así, la comprobación `%~dp0 == %CD%\` PUEDE ser correcta, pero `%~dp0 == %CD%` NO será correcta si estamos en el mismo directorio donde se encuentra el script.
 - Para evitar conflictos, es preferible que los strings con paths utilicen caracteres envolventes a la hora de realizar comprobaciones. Ejemplo: `if /%~dp0/ == /%CD%\/ COMANDO` utiliza / como caracter envolvente en cada string.

## El caracter de escape

Anteponer el caracter de escape ^ a un símbolo de comando hace que el mismo sea tratado como un texto ordinario. Ejemplo: `^\  ^&  ^|  ^>  ^<  ^^ `.

* NOTA: ^ no puede cumplir su función con %, el caracter % tiene un significado especial para los argumentos de línea de comandos y para el comando for. Para tratar un % como un caracter normal se debe escribir %%.

* NOTA: muchos caracteres como \ = ( ) pueden ser utilizados dentro de un string envuelto entre comillas dobles ya que los mismos son caracteres que se pueden encontrar en un filename o path. La única excepción es el caracter %, el cual debe ser puesto dos veces (%%) si quiere utilizarse como un caracter de un filename o path en un sistema de archivos NTFS (donde es válido utilizar el caracter de porcentaje).

El caracter de escape puede ser usado para hacer mas legible los comandos largos. Los comandos largos se pueden dividir en múltiples líneas usando el caracter de escapa junto con CR/LF (Carriage Return + Line Feed, salto de línea en windows):

```batch
echo Visual Studio Express does not include the DIA SDK. ^
You need Visual Studio 2015 or 2017 (Community is free).
```

* NOTA: Hay que tener mucho cuidado con esta técnica de dividir en muchas líneas los comandos largos ya que un espacio entre el caracter ^ y CR/LF provocara una ruptura del comando original. Este error puede ser difícil de ver aun cuando se vea el script con un editor de texto que resalte los espacios y tabulaciones.

Cuando se utiliza pipes, la expresión es parseada dos veces. Primero se parsea antes de que se ejecute el pipe, y una segunda vez después de que el pipe haya sido ejecutado. Así, para hacer un escape de los caracteres en el segundo parseo, es necesario hacer un escape doble. En este ejemplo, se hace un escape del caracter de escape (^^) y un escape del ampersand (^&) para el primer parseo, lo que lleva a un escape del ampersand en el segundo parseo (^&):

```batch
break| echo ^^^&
```
## Comillas dobles y comillas simples

En batch las comillas simples se utilizan solamente en el comando for para procesar un comando (`for /f %%A in ('someCommand') do ...`) o para procesar un string (`for /f "usebackq" %%A in ('some string') do ...`).

Las comillas dobles son útiles para strings o paths que contienen espacios (estos strings o paths suelen ser argumentos pasados por línea de comandos, por ejemplo `MyBatch.cmd "C:\Program Files\My Data File.txt"`). Sin embargo, batch trata una comilla doble como un caracter más (por ejemplo, `%1` retorna `"C:\Program Files\My Data File.txt"`).

* NOTA: no importa si el string está envuelto entre comillas dobles al momento de utilizar las extensiones de las variables que representan los argumentos de linea de comandos.

* NOTA: si el argumento de línea de comandos no se encuentra envuelto en comillas dobles, el mismo será disperso entre las variables %1 %2 %3..., pero %* accederá a todos los parámetros (aun si hay más de 9). Así, si se ejecuta `MyBatch C:\Program Files\My Data File.txt`, para acceder al pathname string `C:\Program Files\My Data File.txt` usar %*.

Para remover las comillas dobles de un string, utilizar cualquiera de los métodos listados en [Remove Quotes from a string - SS64](https://ss64.com/nt/syntax-dequote.html).

## Leer de un archivo y pasar el contenido a una variable

El siguiente ejemplo asigna el contenido de un archivo a una variable:

```batch
set /P GIT= < "%TEMP%\git.loc"
```

## Leer de un archivo y pasar el contenido a un comando (file redirection)

```batch
COMANDO < ARCHIVO
```

## Escribir en un archivo (file redirection)
Para crear un nuevo archivo, o sobrescribir (por sobrescribir me refiero a borrar y escribir desde el principio) uno existente, las siguientes líneas son iguales:

```batch
echo > ARCHIVO TEXTO
echo> ARCHIVO TEXTO
echo>ARCHIVO TEXTO
echo >ARCHIVO TEXTO
echo TEXTO > ARCHIVO
echo TEXTO >ARCHIVO
echo TEXTO 1> ARCHIVO
echo TEXTO 1>ARCHIVO
```

Para agregar una línea a un archivo existente:

```batch
echo >> ARCHIVO TEXTO
```
etc...

* NOTA: batch es sensible a los espacios en blanco. Si TEXTO es, por ejemplo, `  texto` los dos espacios en blanco serán escritos también en el archivo. Esto es una utilidad que puede servir a quienes quieran generar un script para otra herramienta en donde la indentación es necesaria como, por ejemplo, python o make.

## Leer de un archivo y pasar el contenido a otro archivo

El contenido de un archivo puede ser puesto en un nuevo archivo:

```batch
type > ARCHIVO_NUEVO ARCHIVO
type> ARCHIVO_NUEVO ARCHIVO
type>ARCHIVO_NUEVO ARCHIVO
type >ARCHIVO_NUEVO ARCHIVO
type ARCHIVO > ARCHIVO_NUEVO
type ARCHIVO >ARCHIVO_NUEVO
type ARCHIVO 1> ARCHIVO_NUEVO
type ARCHIVO 1>ARCHIVO_NUEVO
```

El contenido de un archivo puede ser agregado a un archivo existente:

```batch
type >> ARCHIVO_EXISTENTE ARCHIVO
```
etc...

## Enviar la salida de un comando como entrada a otro comando (pipe redirection)

Las siguientes líneas son iguales:

```batch
commandA | commandB
commandA|  commandB
commandA |commandB
commandA|commandB
```

## Argumentos de línea de comandos

A diferencia de las variables de entorno (principalmente usadas por scripts batch, creadas, modificadas y borradas en una sesión con SET), las variables de argumento solamente tienen un signo %.

%1 retorna el primer argumento (no el nombre del archivo o script), %2 retorna el segundo, etc. Además, %* retorna todos los argumentos a partir de %1 (%1 %2 %3...).

* NOTA: Únicamente los argumentos del %1 al %9 son referenciables en un script. ¿Cómo leer el resto?, para esto leer [Desplazamiento a través de los argumentos de línea de comandos (reposition, shift)](#Desplazamiento-a-través-de-los-argumentos-de-línea-de-comandos)

%0 es el archivo o script que se ha ejecutado desde la línea de comandos, incluyendo el path que se ha especificado al momento de ejecutar. Por ejemplo, al ejecutar `C:\>foo\bar.cmd` %0 será igual a `foo\bar.cmd`.

Si un argumento n tiene comillas dobles (por ejemplo, "argumento"):

 - %n devolverá el argumento CON las comillas.
 - "%n" devolverá el argumento CON las comillas, envuelto en comillas (por ejemplo, ""argumento"").
 - %~n devolverá el argumento SIN comillas.
 - %~n devolverá el argumento SIN comillas, envuelto en comillas (por ejemplo, "argumento").

Si, por ejemplo, el primer argumento tiene cualquiera de las siguientes formas:

 - clave=valor
 - clave =valor
 - clave= valor
 - clave = valor
 - clave valor

%1 retorna clave y %2 retorna valor.

## Extensiones para los argumentos de línea de comandos

Las siguientes extensiones pueden aplicarse a cualquiera de los %n argumentos. Sin embargo esto solamente suele ser de utilidad para tratar con el archivo actual (%0, el script en ejecución).

 - d -- drive letter
 - p -- path (e.g. \utils\ this includes a trailing \ which will be interpreted as an escape character by some commands)
 - n -- file name without file extension
 - x -- file extension
 - f -- full path (e.g. C:\utils\MyFile.txt)
 - ~ -- remove any surrounding quotes (")
 - a -- file attributes
 - t -- file date/time
 - z -- file size
 - $PATH:n -- Search the PATH environment variable and expand %n to the fully qualified name of the first match found

Así, por ejemplo, %~dp0 es path absoluto al directorio en donde se encuentra nuestro script y %~nx0 es el archivo (y su extensión) en ejecución.

## Desplazamiento a través de los argumentos de línea de comandos

El comando `shift` es utilizado para cambiar la posición de los argumentos de línea de comandos en las variables %0..%9.

Ejemplo: si `%1=-c`, `%2=-p`, `%3=-r`, al ejecutar `shift` obtendremos `%0=-c`, `%1=-p`, `%2=-r`.

Ejemplo: si `%1=--enable-install-doc`, `%2=--pgo`, `%3=--test-marker`, al ejecutar `shift /2` obtendremos `%1=--enable-install-doc`, `%2=--test-marker`.

* NOTA: `shift` no funciona dentro de un grupo de expresiones entre paréntesis (bracketed code). Para esto, guardar los argumentos en variables temporales antes de ejecutar un FOR o cualquier otro bracketed code.

## Evitar que (Command-Echoing)

Command-Echoing es una característica habilitada por defecto en batch la cual imprime cada línea que se ejecuta de un script.

Para deshabilitar esta característica es necesario agregar una línea `echo off`, pero al ejecutar esta línea la misma es impresa en pantalla. Para evitar esto es necesario proceder el comando con un signo @. Así, la primer línea que suele encontrarse en todo script batch es un `@echo off`.

En un script batch, @ es igual a un `echo off` aplicada a la línea actual.

## Evitar que un comando imprima en pantalla

Usar la redirección a nul. Por ejemplo `findstr "^--with-.*-dir$" <file.txt > nul` o `type file.txt| findstr "^--with-.*-dir$" > nul`.

Si además se quiere evitar que los errores se impriman en pantalla, agregar `2> nul`.

## Transferir el control a otra línea de código sin retornar (goto y labels)

Un label es una línea que comienza con `:`, tiene un nombre sin espacios, y termina con espacio, dos puntos, o un salto de línea CR\LF. De esta forma, por ejemplo, el siguiente es un label al que el script puede saltar en cualquier momento:

```batch
:mylabel

```

Para saltar a la posición en donde se encuentra un label, hay que ejecutar cualquiera de estas líneas:

```batch
goto :mylabel
goto mylabel
```

De esta forma, al transferir la ejecución a un label, batch continuará ejecutando las líneas que se encuentren debajo del mismo.

* NOTA: existe un label predefinido llamado `:eof`. Ejecutar `goto :eof` hará que se finalice con la subrutina o script actual

## Transferir el control a otra línea de código y retornar (call y subrutinas)

El comando call nos permite ejecutar un script batch desde otro o llamar una subrutina.

Para llamar un script:

```batch
call [drive:][path]filename [parameters]
```

Para llamar una subrutina:

```batch
call :label [parameters]
```

En una subrutina call pasará el control a la sentencia que se encuentre debajo del label `:label` junto con cualquier parámetro.

Para salir de una subrutina, invocar `goto :eof`. En el final de una subrutina, un `exit \b` dará el control a la posición en donde se ejecutó call (`goto :eof` también puede utilizarse para esto).

Para más información, ver [CALL - SS64](https://ss64.com/nt/call.html)

## Cuidado con las subrutinas y `goto :eof`

 - Una subrutina es una porción de código que será ejecutada por el intérprete batch si se encuentra en el camino de ejecución. Por esta razón las subrutinas suelen ubicarse al final del script.

 - `goto :eof` funciona de diferente manera, dependiendo de cómo se llega a dicha sentencia.
   - En el siguiente ejemplo, la subrutina será ejecutada dos veces (la segunda vez por causa de lo explicado arriba) pero el texto "End Of File" nunca saldrá en pantalla. En la primer ejecución de la subrutina `goto :eof` es interpretado como un retorno del control a la línea que sigue al comando call. Pero en la segunda ejecución `goto :eof` se entiende como una finalización del script.
   ```batch
    @echo off

    echo comienzo de ejecucion...
    call :subrutina
    echo despues de subrutina

    :subrutina
    echo    adentro de subrutina
    goto :eof

    echo End Of File
   ```
   - En el siguiente ejemplo, la subrutina sera ejecutada una sola vez, el texto "End Of File" nunca saldrá en pantalla, ni tampoco el texto `despues de subrutina`. En este caso, `goto :eof` se entiende como una finalización del script. Esto es porque `goto :subrutina` se utiliza con la intención de NO retornar el control a la siguiente línea.
   ```batch
    @echo off

    echo comienzo de ejecucion...
    goto :subrutina
    echo despues de subrutina

    :subrutina
    echo    adentro de subrutina
    goto :eof

    echo End Of File
   ```
## Condición if

Para un if de una única línea, escribirlo en una de las siguientes formas:

```batch
if ITEM1==ITEM2 COMANDO
if ITEM1 == ITEM2 COMANDO
if ITEM1 OPERADOR ITEM2 COMANDO
if ITEM1 OPERADOR ITEM2 (COMANDO)
if ITEM1 OPERADOR ITEM2 (COMANDO) ELSE (COMANDO_ALTERNATIVO)
```

* NOTA: == se usa para una comparación de strings.
* NOTA: == no puede ser utilizado con ELSE.
* NOTA: para la primera y segunda alternativa (donde se utiliza ==), se puede anteponer un NOT para negar la condición: `if NOT ITEM1==ITEM2 COMANDO`.
* NOTA: OPERADOR se usa para una comparación a nivel numérico. Además de enteros, OPERADOR puede comparar números en base hexadecimal y octal con ciertas limitaciones.

OPERADOR puede ser cualquiera de los siguientes operadores:
 - EQU : Equal
 - NEQ : Not equal
 - LSS : Less than <
 - LEQ : Less than or Equal <=
 - GTR : Greater than >
 - GEQ : Greater than or equal >=

* NOTA: No se pueden utilizar > y < como caracteres de OPERADOR ya que los mismos son utilizados para redireccionamiento.

Para hacer una comprobación de strings sin discriminar mayúsculas y minúsculas (case insensitive), usar el parámetro `/I`:

```batch
if /I ITEM1==ITEM2 COMANDO
if /I ITEM1 == ITEM2 COMANDO
if /I ITEM1 OPERADOR ITEM2 COMANDO
if /I ITEM1 OPERADOR ITEM2 (COMANDO)
if /I ITEM1 OPERADOR ITEM2 (COMANDO) ELSE (COMANDO_ALTERNATIVO)
```

Para un if de múltiples líneas, escribirlo en una de las siguientes formas:

```batch
if ITEM1 OPERADOR ITEM2 (
    COMANDO1
    COMANDO2
    COMANDO3
)

if ITEM1 OPERADOR ITEM2 (
    COMANDO1
    COMANDO2
    COMANDO3
) ELSE (
    COMANDO_ALTERNATIVO1
    COMANDO_ALTERNATIVO2
    COMANDO_ALTERNATIVO3
)
```

* NOTA: == no puede utilizarse para un if de múltiples líneas, sin embargo, los operadores de tres letras listados arriba también son capaces de trabajar con strings.

NO existe el operador condicional AND, pero se puede simular encadenando if's. Por ejemplo:

```batch
if ITEM1 OPERADOR ITEM2 if ITEM3 OPERADOR ITEM4 (
    COMANDO1
    COMANDO2
    COMANDO3
)
```
o
```batch
if ITEM1 OPERADOR ITEM2 (
    if ITEM3 OPERADOR ITEM4 (
        COMANDO1
        COMANDO2
        COMANDO3
    )
)
```
o
```batch
if ITEM1 OPERADOR ITEM2 if ITEM3 OPERADOR ITEM4 (
    if ITEM5 OPERADOR ITEM6 if ITEM7 OPERADOR ITEM8 (
        COMANDO1
        COMANDO2
        COMANDO3
    )
)
```
etc...

NO existe el operador condicional OR, pero se puede simular utilizando variables temporales. Por ejemplo:

```batch
set "_tempvar="
If ITEM1==ITEM2 set _tempvar=1
If ITEM1 OPERADOR ITEM2 set _tempvar=1
if %_tempvar% EQU 1 (
    COMANDO1
    COMANDO2
    COMANDO3
)
```

## Condición if con archivos

Para comprobar si un archivo existe, ejecutar cualquiera de las siguientes líneas:

```batch
IF EXIST ARCHIVO COMANDO
IF EXIST ARCHIVO (COMANDO) ELSE (COMANDO_ALTERNATIVO)
```

Para comprobar si un archivo NO existe, ejecutar cualquiera de las siguientes líneas:

```batch
IF NOT EXIST ARCHIVO COMANDO
IF NOT EXIST ARCHIVO (COMANDO) ELSE (COMANDO_ALTERNATIVO)
```

## Condición con el resultado de un comando (errorlevel)

La mayoría de las aplicaciones y utilidades emiten un código de error al terminar su ejecución. Este código se almacena automáticamente en la variable built-in `%ERRORLEVEL%`.

 - Estos códigos de error pueden variar, pero en general el código 0 (false) indica un resultado exitoso.
 - Ejecutar un comando que no existe es un `%ERRORLEVEL%` igual a 9009.

Existen dos métodos para comprobar un errorlevel:

```batch
if errorlevel CODIGO COMANDO
if %ERRORLEVEL% EQU CODIGO COMANDO
```

La primer alternativa debe interpretarse como `if errorlevel >= number`.

* NOTA: Aunque la segunda alternativa es la más lógica y facil de leer, la primera ofrece una compatibilidad hacia atras con Windows 95.

También existe una negación para la primer alternativa:

```batch
if not errorlevel CODIGO COMANDO
```

Aquí la sentencia debe interpretarse como `if errorlevel < number`.

## Condición para comprobar la existencia de un argumento

Para comprobar la existencia de un argumento de línea de comandos usar cualquiera de las siguientes alternativas:

```batch
if [%1]==[] echo value missing
if [%1] EQU [] echo value missing
```

## Condición para comprobar la existencia de una variable

Para comprobar si una variable de entorno se encuentra definida (no es null) usar la siguiente sintaxis:

```batch
if defined VARIABLE COMANDO
```

Para comprobar si una variable de entorno NO se encuentra definida (es null) usar la siguiente sintaxis:

```batch
if not defined VARIABLE COMANDO
```

* NOTA: una variable se encuantra definida cuando tiene cualquier valor (incluso un espacio).

## Where

Equivalente al comando wich en UNIX, encuentra y muestra archivos en un directorio. Por defecto, la busqueda se hace en el directorio actual y en los paths listados en %PATH%. Por ejemplo, `where git` buscara todos los archivos llamados git en el directorio actual y en los directorios listados en %PATH%.

## If y Where

Suelen encontrarse comprobaciones en donde, si una variable que sirve para especificar la ubicacion de un ejecutable no se encuentra definida, entonces se comprueba de forma automatica su ubicacion. El siguiente ejemplo comprueba la ubicacion de git y lo asigna a la variable %GIT%:

```batch
if not exist "%GIT%" where git > "%TEMP%\git.loc" 2> nul && set /P GIT= < "%TEMP%\git.loc" & del "%TEMP%\git.loc"
```

## Bucle For

El comando for acepta diferentes sintaxis en donde cada una trata un caso particular:

 - syntax-FOR-Files
       `FOR %%parameter IN (set) DO command`
 - syntax-FOR-Files-Rooted at Path
       `FOR /R [[drive:]path] %%parameter IN (set) DO command`
 - syntax-FOR-Folders
       `FOR /D %%parameter IN (folder_set) DO command`
 - syntax-FOR-List of numbers
       `FOR /L %%parameter IN (start,step,end) DO command`
 - syntax-FOR-File contents
       `FOR /F ["options"] %%parameter IN (filenameset) DO command`
       `FOR /F ["options"] %%parameter IN ("Text string to process") DO command`
 - syntax-FOR-Command Results
       `FOR /F ["options"] %%parameter IN ('command to process') DO command`

A diferencia de las varaibles de entorno y las variables de argumento, la variable temporal del bucle for se referencia con %% (si se usa en un script batch) y se define con una sola letra (preferentemente G o I).

* NOTA: si el comando for se ejecuta manualmente en una ventana de línea de comandos, usar un solo % y no dos.

Ejemplo:

```batch
for %%I in (%0) do if /%%~dpI/ == /%CD%\/ (
    echo No me ejecutes en mi directorio.
    exit /b 999
)
```

Para más información ver:
 - [FOR - SS64](https://ss64.com/nt/for.html).

## Ejecución de comandos en una misma línea (Run redirection)

```batch
::Run commandA and then run commandB
commandA &  commandB
::Run commandA, if it succeeds then run commandB
commandA && commandB
::Run commandA, if it fails then run commandB
commandA || commandB
::If commandA succeeds run commandB, if it fails commandC
commandA && commandB || commandC
```

## Finalización de la ejecución de un script

`exit /b CODIGO_PARA_ERRORLEVEL` nos permite finalizar la ejecución de un script o subrutina batch sin cerrar la ventana de comandos ().

CODIGO_PARA_ERRORLEVEL es un número que será almacenado en `%ERRORLEVEL%`. 0 indica una ejecución exitosa y 1 o un número mayor indicará un error.

`exit` sin CODIGO_PARA_ERRORLEVEL es igual a `goto :eof` y no cambiará el valor de `%ERRORLEVEL%`.