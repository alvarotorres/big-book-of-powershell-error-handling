# Introducción

El manejo de errores en Windows PowerShell puede ser un tema complejo. El objetivo de este libro -que afortunadamente no es tan "grande" como su nombre lo indica- es ayudar a aclarar algo de esa complejidad y ayudarle a hacer un trabajo mejor y más conciso para manejar errores en sus scripts.

## ¿Qué es el manejo de errores?

Cuando decimos que un script "maneja" un error, significa que reacciona al error haciendo algo distinto del comportamiento predeterminado. En muchos lenguajes de programación y de secuencias de comandos, el comportamiento predeterminado es simplemente mostrar un mensaje de error y fallar inmediatamente. En PowerShell, también se emitirá un mensaje de error, pero a menudo se seguirá ejecutando el código después de que se produzca el error.

El manejo de errores requiere que el autor de la secuencia de comandos anticipe dónde pueden ocurrir y que escriba código para interceptar y analizar dicho errores cuando ocurren. Esto puede ser un tema complejo y a veces frustrante, particularmente en PowerShell. El propósito de este libro es mostrarle las herramientas de manejo de errores que PowerShell pone a su disposición y la mejor forma de usarlas.

## ¿Cómo está organizado este libro?

Después de la introducción, el libro se divide en cuatro secciones. Las dos primeras secciones están escritas asumiendo que usted no sabe nada sobre el manejo de errores de PowerShell, y para proporcionar un sólido contexto sobre el tema. Sin embargo, no hay nada nuevo en estas secciones que no esté cubierto por los archivos de ayuda de PowerShell. Si está bastante familiarizado con el objeto ErrorRecord y los diversos parámetros / variables / declaraciones relacionados con la generación de errores, de informes y de manejo, puede pasar directamente a las secciones 3 y 4.

La sección 3 es una mirada objetiva a las características de manejo de errores de PowerShell, basada en los resultados de algún código de prueba que escribí para entender su funcionamiento. La idea era determinar si existían diferencias funcionales entre enfoques similares para manejar errores ($Error versus ErrorVariable, el uso o no de $\_ en un bloque catch, etc.), lo que generó fuertes opiniones, durante y después los Scripting Games en 2013.

Estas pruebas revelan un par de dificultades, en particular, al hacer uso de ErrorVariable.

La sección 4 resume las cosas dándole una visión más orientada a las tareas de manejo de errores, teniendo en cuenta los hallazgos de la sección 3.
