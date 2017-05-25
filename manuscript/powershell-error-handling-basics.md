# Fundamentos para el manejo de errores en PowerShell

Empecemos por revisar  algunos de los conceptos básicos.

## ErrorRecords y Exceptions

En .NET Framework, sobre el que se construye PowerShell, el reporte de errores se realiza en gran medida lanzando excepciones. Las excepciones son objetos .NET que tienen como tipo base [System.Exception](http://msdn.microsoft.com/en-us/library/system.exception(v=vs.110).aspx). Estos objetos de excepción contienen suficiente información para comunicar todos los detalles del error a una aplicación de .NET Framework (el tipo de error que ocurrió, un seguimiento de pila de llamadas del método que condujo al error, etc.) que por sí solo no es suficiente información para proporcionar a un script de PowerShell. Por eso, PowerShell tiene su propio seguimiento de la pila de scripts y de llamadas de función de las que .NET Framework no sabe nada. También es importante saber qué objetos generaron errores, ya que una única sentencia o tubería (pipeline) es capaz de producir múltiples errores.

Por estas razones, PowerShell expone el objeto ErrorRecord. ErrorRecord contienen una excepción .NET, junto con varias otras piezas de información específica de PowerShell. Por ejemplo, la figura 1.1 muestra cómo acceder a las propiedades TargetObject, CategoryInfo e InvocationInfo de un objeto ErrorRecord; que proporcionan información útil para la lógica de manejo de errores de su script.

![image003.png](images/image003.png)

Figura 1.1: Algunas de las propiedades más útiles del objeto ErrorRecord.

## Terminating versus Non-Terminating Errors

PowerShell is an extremely _expressive_ language. This means that a single statement or pipeline of PowerShell code can perform the work of hundreds, or even thousands of raw CPU instructions. For example:

Get-Content .\computers.txt | Restart-Computer

This small, 46-character PowerShell pipeline opens a file on disk, automatically detects its text encoding, reads the text one line at a time, connects to each remote computer named in the file, authenticates to that computer, and if successful, restarts the computer. Several of these steps might encounter errors; in the case of the Restart-Computer command, it may succeed for some computers and fail for others.

For this reason, PowerShell introduces the concept of a Non-Terminating error. A Non-Terminating error is one that does not prevent the command from moving on and trying the next item on a list of inputs; for example, if one of the computers in the computers.txt file is offline, that doesn't stop PowerShell from moving on and rebooting the rest of the computers in the file.

By contrast, a Terminating error is one that causes the entire pipeline to fail. For example, this similar command fetches the email addresses associated with Active Directory user accounts:
```
Get-Content .\users.txt |
Get-ADUser -Properties mail |
Select-Object -Property SamAccountName,mail
```
In this pipeline, if the Get-ADUser command can't communicate with Active Directory at all, there's no reason to continue reading lines from the text file or attempting to process additional records, so it will produce a Terminating error. When this Terminating error is encountered, the entire pipeline is immediately aborted; Get-Content will stop reading lines, and close the file.

It's important to know the distinction between these types of errors, because your scripts will use different techniques to intercept them. As a general rule, most errors produced by Cmdlets are non-terminating (though there are a few exceptions, here and there.)

