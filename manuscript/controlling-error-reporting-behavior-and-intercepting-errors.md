# Controlando el comportamiento de los errores

Esta sección muestra brevemente cómo usar cada una de las declaraciones, variables y parámetros de PowerShell que están relacionados con el reporte o manejo de errores.

## La variable $Error

$Error es una variable global automática en PowerShell que siempre contiene un ArrayList de cero o más objetos ErrorRecord. A medida que se producen nuevos errores, se agregan al principio de esta lista, por lo que siempre se puede obtener información sobre el error más reciente utilizando  $Error[0]. Los errores Terminating y Non-Terminating se incluirán en esta lista.

Aparte de acceder a los objetos de la lista con la sintaxis de matriz, hay otras dos tareas comunes que se realizan con la variable $Error: Se puede comprobar cuántos errores están actualmente en la lista utilizando la propiedad $Error.Count y puede eliminar todos los errores de la lista con el método $Error.Clear(). Por ejemplo:

![image004.png](images/image004.png)

Figura 2.1: Utilizando $Error para acceder a la información de error, verificar el recuento y borrar la lista.

Si está planeando hacer uso de la variable $Error en sus scripts, tenga en cuenta que puede contener información sobre errores que ocurrieron en la sesión actual de PowerShell, pero antes de que se iniciara la ejecución de su secuencia de comandos. Algunas personas consideran una mala práctica borrar la variable $Error dentro de un script. Como se trata de una variable global para la sesión de PowerShell, la persona que llamó a su secuencia de comandos podría revisar el contenido de $Error después de que su comando haya terminado la ejecución..

## ErrorVariable

El parámetro común ErrorVariable proporciona una alternativa al uso de la colección $Error anterior. A diferencia de $Error, ErrorVariable sólo contendrá los errores que se produjeron desde el comando que se está llamando, en lugar de tener potencialmente errores de otras partes den la sesión PowerShell. Esto también evita tener que borrar el contenido de $Error (y evitar los problemas que esto podría ocasionar).

Cuando se utiliza ErrorVariable, si desea anexar a la variable de error en lugar de sobrescribirla, coloque un signo + delante del nombre de la variable. Tenga en cuenta que no se utiliza un signo de moneda cuando pasa un nombre de variable al parámetro ErrorVariable, pero si utiliza el signo de moneda más adelante cuando comprueba su valor.

La variable asignada al parámetro ErrorVariable nunca será nula. Si no se produjeron errores, contendrá un objeto ArrayList con un recuento de 0, como se ve en la figura 2.2:

![image005.png](images/image005.png)

Figure 2.2: Demonstrating the use of the ErrorVariable parameter.

## $MaximumErrorCount

By default, the $Error variable can only contain a maximum of 256 errors before it starts to lose the oldest ones on the list. You can adjust this behavior by modifying the $MaximumErrorCount variable.

## ErrorAction and $ErrorActionPreference

There are several ways you can control PowerShell's handling / reporting behavior. The ones you will probably use most often are the ErrorAction common parameter and the $ErrorActionPreference variable.

The ErrorAction parameter can be passed to any Cmdlet or Advanced Function, and can have one of the following values: Continue (the default), SilentlyContinue, Stop, Inquire, Ignore (only in PowerShell 3.0 or later), and Suspend (only for workflows; will not be discussed further here.) It affects how the Cmdlet behaves when it produces a non-terminating error.

- The default value of Continue causes the error to be written to the Error stream and added to the $Error variable, and then the Cmdlet continues processing.
- A value of SilentlyContinue only adds the error to the $Error variable; it does not write the error to the Error stream (so it will not be displayed at the console).
- A value of Ignore both suppresses the error message and does not add it to the $Error variable. This option was added with PowerShell 3.0.
- A value of Stop causes non-terminating errors to be treated as terminating errors instead, immediately halting the Cmdlet's execution. This also enables you to intercept those errors in a Try/Catch or Trap statement, as described later in this section.
- A value of Inquire causes PowerShell to ask the user whether the script should continue or not when an error occurs.

The $ErrorActionPreference variable can be used just like the ErrorAction parameter, with a couple of exceptions: you cannot set $ErrorActionPreference to either Ignore or Suspend. Also, $ErrorActionPreference affects your current scope in addition to any child commands you call; this subtle difference has the effect of allowing you to control the behavior of errors that are produced by .NET methods, or other causes such as PowerShell encountering a "command not found" error.

Figure 2.3 demonstrates the effects of the three most commonly used $ErrorActionPreference settings.

![image006.png](images/image006.png)

Figure 2.3: Behavior of $ErrorActionPreference

## Try/Catch/Finally

The Try/Catch/Finally statements, added in PowerShell 2.0, are the preferred way of handling _terminating_ errors. They cannot be used to handle non-terminating errors, unless you force those errors to become terminating errors with ErrorAction or $ErrorActionPreference set to Stop.

To use Try/Catch/Finally, you start with the "Try" keyword followed by a single PowerShell script block. Following the Try block can be any number of Catch blocks, and either zero or one Finally block. There must be a minimum of either one Catch block or one Finally block; a Try block cannot be used by itself.

The code inside the Try block is executed until it is either complete, or a terminating error occurs. If a terminating error does occur, execution of the code in the Try block stops. PowerShell writes the terminating error to the $Error list, and looks for a matching Catch block (either in the current scope, or in any parent scopes.) If no Catch block exists to handle the error, PowerShell writes the error to the Error stream, the same thing it would have done if the error had occurred outside of a Try block.

Catch blocks can be written to only catch specific types of Exceptions, or to catch all terminating errors. If you do define multiple catch blocks for different exception types, be sure to place the more specific blocks at the top of the list; PowerShell searches catch blocks from top to bottom, and stops as soon as it finds one that is a match.

If a Finally block is included, its code is executed after both the Try and Catch blocks are complete, regardless of whether an error occurred or not. This is primarily intended to perform cleanup of resources (freeing up memory, calling objects' Close() or Dispose() methods, etc.)

Figure 2.4 demonstrates the use of a Try/Catch/Finally block:

![image007.png](images/image007.png)

Figure 2.4: Example of using try/catch/finally.

Notice that "Statement after the error" is never displayed, because a terminating error occurred on the previous line. Because the error was based on an IOException, that Catch block was executed, instead of the general "catch-all" block below it. Afterward, the Finally executes and changes the value of $testVariable.

Also notice that while the Catch block specified a type of [System.IO.IOException], the actual exception type was, in this case, [System.IO.DirectoryNotFoundException]. This works because DirectoryNotFoundException is _inherited_ from IOException, the same way all exceptions share the same base type of System.Exception. You can see this in figure 2.5:

![image008.png](images/image008.png)

Figure 2.5: Showing that IOException is the Base type for DirectoryNotFoundException

## Trap

Trap statements were the method of handling terminating errors in PowerShell 1.0. As with Try/Catch/Finally, the Trap statement has no effect on non-terminating errors.

Trap is a bit awkward to use, as it applies to the entire scope where it is defined (and child scopes as well), rather than having the error handling logic kept close to the code that might produce the error the way it is when you use Try/Catch/Finally. For those of you familiar with Visual Basic, Trap is a lot like "On Error Goto". For that reason, Trap statements don't see a lot of use in modern PowerShell scripts, and I didn't include them in the test scripts or analysis in Section 3 of this ebook.

For the sake of completeness, here's an example of how to use Trap:

![image009.png](images/image009.png)

Figure 2.6: Use of the Trap statement

As you can see, Trap blocks are defined much the same way as Catch blocks, optionally specifying an Exception type. Trap blocks may optionally end with either a Break or Continue statement. If you don't use either of those, the error is written to the Error stream, and the current script block continues with the next line after the error. If you use Break, as seen in figure 2.5, the error is written to the Error stream, and the rest of the current script block is not executed. If you use Continue, the error is not written to the error stream, and the script block continues execution with the next statement.

## The $LASTEXITCODE Variable

When you call an executable program instead of a PowerShell Cmdlet, Script or Function, the $LASTEXITCODE variable automatically contains the process's exit code. Most processes use the convention of setting an exit code of zero when the code finishes successfully, and non-zero if an error occurred, but this is not guaranteed. It's up to the developer of the executable to determine what its exit codes mean.

Note that the $LASTEXITCODE variable is only set when you call an executable directly, or via PowerShell's call operator (&) or the Invoke-Expression cmdlet. If you use another method such as Start-Process or WMI to launch the executable, they have their own ways of communicating the exit code to you, and will not affect the current value of $LASTEXITCODE.

![image010.png](images/image010.png)

Figure 2.7: Using $LASTEXITCODE.

## The $? Variable

The $? variable is a Boolean value that is automatically set after each PowerShell statement or pipeline finishes execution. It should be set to True if the previous command was successful, and False if there was an error. If the previous command was a call to a native exe, $? will be set to True if the $LASTEXITCODE variable equals zero, and False otherwise. When the previous command was a PowerShell statement, $? will be set to False if any errors occurred (even if ErrorAction was set to SilentlyContinue or Ignore.)

Just be aware that the value of this variable is reset after every statement. You must check its value immediately after the command you're interested in, or it will be overwritten (probably to True). Figure 2.8 demonstrates this behavior. The first time $? is checked, it is set to False, because the Get-Item encountered an error. The second time $? was checked, it was set to True, because the previous command was successful; in this case, the previous command was "$?" from the first time the variable's value was displayed.

![image011.png](images/image011.png)

Figure 2.8: Demonstrating behavior of the $? variable.

The $? variable doesn't give you any details about what error occurred; it's simply a flag that something went wrong. In the case of calling executable programs, you need to be sure that they return an exit code of 0 to indicate success and non-zero to indicate an error before you can rely on the contents of $?.

## Summary

That covers all of the techniques you can use to either control error reporting or intercept and handle errors in a PowerShell script. To summarize:

* To intercept and react to non-terminating errors, you check the contents of either the automatic $Error collection, or the variable you specified as the ErrorVariable. This is done after the command completes; you cannot react to a non-terminating error before the Cmdlet or Function finishes its work.
* To intercept and react to terminating errors, you use either Try/Catch/Finally (preferred), or Trap (old and not used much now.) Both of these constructs allow you to specify different script blocks to react to different types of Exceptions.
* Using the ErrorAction parameter, you can change how PowerShell cmdlets and functions report non-terminating errors. Setting this to Stop causes them to become terminating errors instead, which can be intercepted with Try/Catch/Finally or Trap.
* $ErrorActionPreference works like ErrorAction, except it can also affect PowerShell's behavior when a terminating error occurs, even if those errors came from a .NET method instead of a cmdlet.
* $LASTEXITCODE contains the exit code of external executables. An exit code of zero usually indicates success, but that's up to the author of the program.
* $? can tell you whether the previous command was successful, though you have to be careful about using it with external commands, if they don't follow the convention of using an exit code of zero as an indicator of success. You also need to make sure you check the contents of $? immediately after the command you are interested in.



