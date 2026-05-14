Zj-58, Zj-80 y otras impresoras de tickets
==========================================

Filtro CUPS para impresoras térmicas económicas como Zijiang ZJ-58, XPrinter XP-58, JZ-80 con cortador, Epson TM-T20, y posiblemente cualquier otra impresora que entienda comandos ESC/POS.

Originalmente fue un filtro de ingeniería inversa para Zijiang ZJ-58 con su PPD específico, pero luego se descubrió que realmente funciona con muchas otras impresoras económicas de 58mm, como Xprinter XP-58.

Características
---------------

**Cortador**. Si está disponible, puede invocarse después de cada página o después de todo el trabajo.

Soporte para **2 cajones de efectivo**, cada uno puede abrirse antes o después de un trabajo de impresión completo. La longitud de los impulsos para accionar los cajones puede personalizarse.

**Avances en blanco**: después del trabajo impreso, o después de cada página, el papel puede avanzarse entre 3 y 45 mm adicionales en pasos de 3 mm.

**Saltar cola en blanco**: si imprimes un trozo pequeño de datos en un rollo grande, el filtro puede determinar la 'cola' vacía de tu impresión y evitar alimentar papel extra. Es decir, imprimir un fragmento de 20 mm en un papel de 58x210 mm imprimirá solo los 20 mm realmente usados y omitirá el resto de la cola.

Detalles
--------

 * La impresora se inicializa con 'ESC @'.
 * El cortador funciona con 'GS V \1'.
 * El ráster se imprime con 'GS v 0 <x> <y>'.
 * El avance de línea se hace con 'ESC J <N>'.

Al imprimir, el controlador verifica cada línea ráster si está completamente vacía — y si es así, usa el comando de avance en lugar de la impresión ráster real.

El número de modelo del PPD contiene la cantidad de píxeles del cabezal de la impresora. El filtro extrae este valor y así es como se diferencian 58 mm de 80 mm (también puedes personalizarlo con tus propios valores sin necesidad de recompilar el filtro).

Los ajustes de impresión se pasan y almacenan dentro de un trabajo de impresión, en el encabezado de cada página. Esto es diferente al enfoque anterior donde se analizaba el PPD cada vez.

Dimensiones
-----------

Una impresora de 58mm típicamente tiene 384 píxeles activos, colocados en una fila de 48 mm de longitud. Eso son exactamente 8 píxeles/mm. Calculando de píxeles/mm a DPI obtienes aproximadamente 203 DPI (25.4 mm/in * 8 px/mm = 203.2 DPI). En términos de puntos (1 pt = 1/72 in), un papel de 58 mm tiene 164.4 pt de ancho, y el área de impresión de 48 mm es 136 pt. Así que, para una página de 164 pt, tenemos primero un margen izquierdo de 14 pt, luego 136 pt de área de impresión y finalmente 14 pt de margen derecho. Estas son limitaciones físicas y no se pueden modificar.

Por lo tanto, para personalizar el PPD para una impresora con otras dimensiones necesitas conocer estos valores: **resolución** (en DPI), **cantidad de píxeles**, y también el **ancho del papel** físico y el **área de impresión** expresados en puntos. La resolución y los márgenes están en regiones separadas del PPD, el ancho del papel se usa al definir tamaños de medio; la cantidad de píxeles se coloca en el campo 'modelnum' del PPD.

Compilación e instalación
=========================

Necesitas herramientas de compilación, CMake y cups-devel.

Puede lograrse, por ejemplo, ejecutando:
```
sudo apt install build-essential cmake libcups2-dev libcupsimage2-dev
```

Además, si quieres compilar tu propio PPD, es muy recomendable tener disponible el compilador `ppdc`.
La compilación se realiza fuera del directorio fuente.

```
  mkdir build && cd build && cmake /ruta/a/las/fuentes
  cmake --build .
```

La instalación implica reiniciar el servicio CUPS y también colocar los archivos compilados en los directorios del sistema, lo que requiere permisos de administrador.

```
  sudo make install
```

El script de CMake tiene escenarios de instalación tanto para Linux como para Mac OS X.

*¡IMPORTANTE!* Si actualizas desde una versión anterior del filtro, DEBES reconfigurar manualmente tu impresora anterior y seleccionar explícitamente el archivo PPD de aquí.
CUPS por defecto no lo hará, y el PPD anterior con este filtro fallará e incluso hará fallar el trabajo. Por lo tanto, si anteriormente usabas el controlador de este repositorio y empezó a fallar después de instalar esta versión, debes ir a las preferencias de la impresora y seleccionar el controlador 'nuevo' en lugar del anterior almacenado en caché por CUPS.

Configuración
=============

Generalmente no es necesario, pero puedes encontrar algunos valores ejecutando `ccmake` en lugar de `cmake` en tu carpeta de compilación.

Depuración
----------

La versión de depuración del filtro puede obtenerse usando `cmake -DCMAKE_BUILD_TYPE=Debug`, o cambiando el mismo parámetro en la interfaz TUI de `ccmake`. La versión de depuración realizará salida de diagnóstico en el archivo señalado por el parámetro `DEBUGFILE`. En Mac OS, donde CUPS está fuertemente aislado (sandbox), este archivo es seleccionado por el filtro y no puede personalizarse (sin embargo, puedes dirigir tu visor de texto a esa ubicación). Esto se debe a que los filtros de CUPS son muy limitados y no pueden escribir en cualquier carpeta deseable.

Empaquetado
===========

Cualquier empaquetado soportado por CMake debería funcionar. Sin embargo, ten en cuenta que internamente 'empaquetar' es lo mismo que 'instalar en una carpeta personalizada y luego empaquetarlo'. Y dado que la instalación implica cosas como cambiar permisos/propietario y reiniciar el servicio CUPS, esto también ocurre al empaquetar (quizás sea posible evitarlo, pero no por ahora). Por lo tanto, `make package` y `cpack .` ambos requieren permisos `sudo` y reiniciarán CUPS como efecto secundario (aunque los archivos no se instalarán, sino que se empaquetarán).

También puedes obtener un paquete debian ejecutando:
```
  sudo cpack -G DEB
```
