# Poniéndolo todo junto
Ahora que hemos examinado todas las herramientas de manejo de errores e identificado algunos posibles escenarios de "engañosos", he aquí algunos consejos y ejemplos de cómo abordar el manejo de errores en sus propios scripts.

## Supresión de errores (no haga esto)

Hay ocasiones en las que puede “procesar” un error sin la intención de manejarlo. En realidad, las situaciones válidas para estos escenarios son pocas. Procure no establecer ErrorAction o $ErrorActionPreference  en SilentlyContinue a menos que tenga la intención de examinar y verificar cada posible error, usted mismo más adelante en su código. Utilizar un bloque Try/Catch con un bloque Catch vacío equivale a la misma cosa. Por lo general esto no es lo correcto.

Es mejor al menos mostrar al usuario la salida de error por defecto en la consola, que tener un comando que falle sin indicación alguna de que algo salió mal.

## Uso de la variable $? (úselo bajo su propio riesgo)

La variable $? parece una buena idea al principio, pero hay muchas cosas que podrían salir mña como para simplemente confiar en esta variable en un script de producción. Por ejemplo, si el error es generado por un comando que está entre paréntesis o una sub-expresión, la variable $? se establecerá en true en lugar de false:

![image015.png](images/image015.png)

Figura 4.1: Falsos positivos con la variable $?

## Determinar qué tipos de errores puede producir un comando

Antes de que pueda decidir la mejor manera de manejar los errores de un comando en particular, a menudo necesitará saber qué tipo de errores puede producir. ¿Terminating o Non-Terminating? ¿Cuáles son los tipos de excepción que se pueden producir? Desafortunadamente, la documentación del Cmdlet de PowerShell no proporciona esta información, por lo que necesita recurrir a algún tipo de prueba y error. Aquí hay un ejemplo de cómo puede averiguar si los errores de un Cmdlet son Terminating or Non-Terminating:

![image016.png](images/image016.png)

Figura 4.2: Identificación de errores

Irónicamente, este era un lugar práctico tanto para usar la sentencia Trap como para establecer $ErrorActionPreference a SilentlyContinue, cosas que casi nunca haría en un script de producción. Como se puede ver en la figura 4.2, Get-Acl produce excepciones Terminating cuando el archivo existe, pero el Cmdlet no puede leer el ACL. Get-Item y Get-Acl producen errores Non-Terminating si el archivo no existe.

Pasar por este tipo de ensayo y error puede ser un proceso que consume mucho tiempo, sin embargo, es necesario que conozca las diferentes formas en que un comando puede fallar y, a continuación, reproducir esas condiciones para ver si el error resultante era Terminating o Non-Terminating. Como resultado de lo molesto que puede ser, además de este libro electrónico, el repositorio de Github contendrá una hoja de cálculo con una lista de errores Terminating conocidos de algunos Cmdlets. Será un documento “vivo”, posiblemente convertido en un wiki en algún momento, pero probablemente nunca será una referencia completa, debido a la gran cantidad de Cmdlets de PowerShell que existen por ahí, aunque esto es mucho mejor que nada.

Además de saber si los errores son Terminating o Non- Terminating, es posible que también desee conocer qué tipos de excepciones se producen. La figura 4.3 muestra cómo puede enumerar los tipos de excepción que están asociados con diferentes tipos de errores. Cada objeto de excepción puede contener opcionalmente una InnerException, y puede usar cualquiera de ellos en un bloque Catch o Trap:

![image017.png](images/image017.png)

Figura 4.3: Visualización de los tipos de Excepciones y de cualquier InnerException.

## Dealing with Terminating Errors

This is the easy part. Just use Try/Catch, and refer to either $\_ or $error[0] in your Catch blocks to get information about the terminating error.

## Dealing with Non-Terminating Errors

I tend to categorize commands that can produce Non-Terminating errors (Cmdlets, functions and scripts) in one of three ways: Commands that only need to process a single input object, commands that can only produce Non-Terminating errors, and commands that could produce a Terminating or Non-Terminating error. I handle each of these categories in the following ways:

If the command only needs to process a single input object, as in figure 4.4, I use ErrorAction Stop and handle errors with Try/Catch. Because the cmdlet is only dealing with a single input object, the concept of a Non-Terminating error is not terribly useful anyway.

![image018.png](images/image018.png)

Figure 4.4: Using Try/Catch and ErrorAction Stop when dealing with a single object.

If the command should only ever produce Non-Terminating errors, I use ErrorVariable. This category is larger than you'd think; most PowerShell cmdlet errors are Non-Terminating:

![image019.png](images/image019.png)

Figure 4.5: Using ErrorVariable when you won't be annoyed by its behavior arising from Terminating errors.

When you're examining the contents of your ErrorVariable, remember that you can usually get useful information about what failed by looking at an ErrorRecord's CategoryInfo.Activity property (which cmdlet produced the error) and TargetObject property (which object was it processing when the error occurred). However, not all cmdlets populate the ErrorRecord with a TargetObject, so you'll want to do some testing ahead of time to determine how useful this technique will be. If you find a situation where a cmdlet should be telling you about the TargetObject, but doesn't, consider changing your code structure to process one object at a time, as in figure 4.4. That way, you'll already know what object is being processed.

A trickier scenario arises if a particular command might produce either Terminating or Non-Terminating errors. In those situations, if it's practical, I try to change my code to call the command on one object at a time. If you find yourself in a situation where this is not desirable (though I'm hard pressed to come up with an example), I recommend the following approach to avoid ErrorVariable's quirky behavior and also avoid calling $error.Clear():

![image020.png](images/image020.png)

Figure 4.6: Using $error without calling Clear() and ignoring previously-existing error records.

As you can see, the structure of this code is almost the same as when using the ErrorVariable parameter, with the addition of a Try block around the offending code, and the use of the $previousError variable to make sure we're only reacting to new errors in the $error collection. In this case, I have an empty Catch block, because the terminating error (if one occurs) is going to be also added to $error and handled in the foreach loop anyway. You may prefer to handle the terminating error in the Catch block and non-terminating errors in the loop; either way works.

## Calling external programs

When you need to call an external executable, most of the time you'll get the best results by checking $LASTEXITCODE for status information; however, you'll need do your homework on the program to make sure it returns useful information via its exit code. There are some odd executables out there that always return 0, regardless of whether they encountered errors.

If an external executable writes anything to the StdErr stream, PowerShell sometimes sees this and wraps the text in an ErrorRecord, but this behavior doesn't seem to be consistent. I'm not sure yet under what conditions these errors will be produced, so I tend to stick with $LASTEXITCODE when I need to tell whether an external command worked or not.

