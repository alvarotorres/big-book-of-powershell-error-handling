#  Análisis de los resultados de las pruebas de manejo de errores

Como se mencionó en la introducción, el código de prueba y sus archivos de salida están disponibles para su descarga. Vea la sección "acerca de", al comienzo de este libro, para conocer la ubicación. Son un montón de datos, no muy bien formateados en un documento de Word, por lo que no serán incluidos en el contenido de los archivos en este libro. Si te cuestionas acerca de  cualquiera de los análisis o conclusiones que he presentado en esta sección, te animo a descargar y revisar tanto el código como los archivos de resultados.

El código de prueba consta de dos archivos. El primero es un módulo de PowerShell (ErrorHandlingTestCommands.psm1) que contiene un Cmdlet, una clase .NET y varias funciones avanzadas para producir errores Terminating y Non-Terminating a demanda, o para probar el comportamiento de PowerShell cuando se producen tales errores. El segundo archivo es el script ErrorTests.ps1, que importa el módulo, llama a sus comandos con varios parámetros y produce la salida que fue redirigida (incluyendo la secuencia de errores) a los tres archivos de resultados: ErrorTests.v2.txt, ErrorTests.v3. Txt y ErrorTests.v4.txt.

Hay tres secciones principales en el script ErrorTests.ps1. La primera sección llama a los comandos para generar errores Terminating y Non-Terminating, y envía información sobre el contenido de $_ (en bloques Catch solamente), $Error y ErrorVariable. Estas pruebas tenían como objetivo responder a las siguientes preguntas:

- Cuando se trata sólo de errores Non-Terminating, ¿hay diferencias entre cómo $Error y ErrorVariable presentan la información acerca de los errores que ocurrieron? ¿Hay alguna diferencia si los errores provienen de un Cmdlet o función avanzada?

- Cuando se utiliza un bloque Try/Catch, ¿Hay diferencias en el comportamiento entre la forma en como $Error, ErrorVariable y $_ proporcionan información sobre el error Terminating que se produjo? ¿Hay alguna diferencia si los errores proceden de un Cmdlet, función avanzada o un método .NET?

- Cuando se producen errores Non-Terminating además del error, ¿Hay diferencias entre cómo $Error y ErrorVariable presentan la información? ¿Hay alguna diferencia cuando los errores provienen de un Cmdlet o función avanzada?

- En las pruebas anteriores, ¿Hay alguna diferencia entre un error Terminating que se produjo normalmente, en comparación con un error Non-Terminating que se produjo cuando ErrorAction o $ErrorActionPreference se establecieron a Stop?

La segunda sección consiste en algunas pruebas para determinar si ErrorAction o $ ErrorActionPreference afectan a los errores Terminating, o sólo a los errores Non-Terminating.

La sección final prueba cómo se comporta PowerShell cuando encuentra errores Terminating no controlados de cada origen posible (un Cmdlet que utiliza PSCmdlet.ThrowTerminatingError(), una función avanzada utiliza la sentencia Throw de PowerShell, un método .NET que genera una excepción, un Cmdlet o una Función avanzada que produce errores Non-Terminating cuando ErrorAction se establece en Stop en un comando desconocido).

Los resultados de todas las pruebas fueron idénticos en PowerShell 3.0 y 4.0. Powershell 2.0 tuvo un par de diferencias, que veremos en el análisis.

## Interceptando errores Non-Terminating

Comencemos hablando de errores Non-Terminating.

### ErrorVariable versus $Error

Cuando se trata de errores Non-Terminating, sólo hay una diferencia entre $Error y ErrorVariable: el orden de los errores en las listas se invierte. El error más reciente que se produce siempre se encuentra al principio de la variable $Error (índice cero) mientras que el error más reciente se encuentra al final de ErrorVariable.

## Interceptando errores Terminating

Esta es la verdadera “carne de la tarea”: Trabajar con errores Terminating, o excepciones.

### $\_

Al principio de un bloque Catch, la variable $\_ siempre se refiere a un objeto ErrorRecord para el error Terminating, independientemente de cómo se produjo ese error.

### $Error

Al principio de un bloque Catch, $Error[0] siempre se refiere a un objeto ErrorRecord para el error Terminating, independientemente de cómo se produjo ese error.

### ErrorVariable

Aquí, las cosas empiezan a complicarse. Cuando un error Terminating  se produce por un Cmdlet o una función y está utilizando ErrorVariable, la variable contendrá algunos elementos inesperados y los resultados son bastante diferentes en las distintas pruebas realizadas:

- Cuando se llama a una función avanzada que genera un error Terminating, ErrorVariable contiene dos objetos de ErrorRecord idénticos para el error. Además, si está ejecutando PowerShell 2.0, estos registros de errores son seguidos por dos objetos idénticos de tipo System.Management.Automation.RuntimeException. Estos objetos RuntimeException contienen una propiedad ErrorRecord, que hace referencia a los objetos ErrorRecord idénticos al par que también figuraba en la lista ErrorVariable. Los objetos adicionales RuntimeException no están presentes en PowerShell 3.0 o posterior.

- Cuando se llama a un Cmdlet que genera un error Terminating, ErrorVariable contiene un solo registro, pero no es un objeto ErrorRecord. En su lugar, es una instancia de System.Management.Automation.CmdletInvocationException. Como los objetos RuntimeException mencionados en el último punto, CmdletInvocationException tiene una propiedad ErrorRecord y esa propiedad se refiere al objeto ErrorRecord que se esperaba que estuviera contenido en la lista ErrorVariable.

- Cuando se llama a una función avanzada con ErrorAction establecido a Stop, ErrorVariable contiene un objeto del tipo System.Management.Automation.ActionPreferenceStopException, seguido por dos objetos de ErrorRecord idénticos. Como con los tipos RuntimeException y CmdletInvocationException, ActionPreferenceStopException todos contiene una propiedad ErrorRecord, que se refiere a un objeto ErrorRecord que es idéntico a los dos que se incluyeron directamente en la lista ErrorVariable. Además, si se ejecuta PowerShell 2.0, hay dos objetos más idénticos al tipo ActionPreferenceStopException, para un total de 5 entradas relacionadas con el mismo error de Terminating.

- Cuando se llama a un Cmdlet con ErrorAction establecido a Stop, ErrorVariable contiene un único objeto del tipo System.Management.Automation.ActionPreferenceStopException. La propiedad ErrorRecord de este objeto ActionPreferenceStopException contiene el objeto ErrorRecord que se esperaba que estuviera directamente en la lista ErrorVariable.

## Effects of setting ErrorAction or $ErrorActionPreference

When you execute a Cmdlet or Advanced Function and set the ErrorAction parameter, it affects the behavior of all non-terminating errors. However, it also appears to affect terminating errors produced by the Throw statement in an Advanced Function (though not terminating errors coming from Cmdlets via the PSCmdlet.ThrowTerminatingError() method.)

If you set the $ErrorActionPreference variable before calling the command, its value affects both terminating and non-terminating errors.

This is undocumented behavior; PowerShell's help files state that both the preference variable and parameter should only be affecting non-terminating errors.

## How PowerShell behaves when it encounters unhandled terminating errors

This section of the code proved to be a bit annoying to test, because if the parent scope (the script) handled the errors, it affected the behavior of the code inside the functions. If the script scope didn't have any error handling, then in many cases, the unhandled error actually aborted the script as well. As a result, the ErrorTests.ps1 script and the text files containing its output are written to only show you the cases where a terminating error occurs, but the function still moves on and executes the next command.

If you want to run the full battery of tests on this behavior, import the ErrorHandlingTests.psm1 module and execute the following commands manually at a PowerShell console. Because you're executing them one at a time, you won't run into an issue with some of the commands failing to execute because of a previous unhandled error, the way you would if these were all in a script.
```
Test-WithoutRethrow -Cmdlet -Terminating

Test-WithoutRethrow -Function -Terminating

Test-WithoutRethrow -Cmdlet -NonTerminating

Test-WithoutRethrow -Function -NonTerminating

Test-WithoutRethrow -Method

Test-WithoutRethrow -UnknownCommand
```
There is also a Test-WithRethrow function that can be called with the same parameters, to demonstrate that the results are consistent across all 6 cases when you handle each error and choose whether to abort the function.

### PowerShell continues execution after a terminating error is produced by:

- Terminating errors from Cmdlets.
- .NET Methods that throw exceptions.
- PowerShell encountering an unknown command.

### PowerShell aborts execution when a terminating error is produced by:

- Functions that use the Throw statement.
- Any non-terminating error in conjunction with ErrorAction Stop.
- Any time $ErrorActionPreference is set to Stop in the caller's scope.

In order to achieve consistent behavior between these different sources of terminating errors, you can put commands that might potentially produce a terminating error into a Try block. In the Catch block, you can decide whether to abort execution of the current script block or not. Figure 3.1 shows an example of forcing a function to abort when it hits a terminating exception from a Cmdlet (a situation where PowerShell would normally just continue and execute the "After terminating error." statement), by re-throwing the error from the Catch block. When Throw is used with no arguments inside of a Catch block, it passes the same error up to the parent scope.

![image013.png](images/image013.png)

Figure 3.1: Re-throwing a terminating error to force a function to stop execution.

## Conclusions

For non-terminating errors, you can use either $Error or ErrorVariable without any real headaches. While the order of the ErrorRecords is reversed between these options, you can easily deal with that in your code, assuming you consider that to be a problem at all. As soon as terminating errors enter the picture, however, ErrorVariable has some very annoying behavior: it sometimes contains Exception objects instead of ErrorRecords, and in many cases, has one or more duplicate objects all relating to the terminating error. While it is possible to code around these quirks, it really doesn't seem to be worth the effort when you can easily use $\_ or $Error[0].

When you're calling a command that might produce a terminating error and you do not handle that error with a Try/Catch or Trap statement, PowerShell's behavior is inconsistent, depending on how the terminating error was generated. In order to achieve consistent results regardless of what commands you're calling, place such commands into a Try block, and choose whether or not to re-throw the error in the Catch block.

![image014.png](images/image014.png)


