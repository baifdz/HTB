Your boss was so impressed with your research skills that you've been "rewarded" with a new task: malware analysis. Your mission is to investigate a piece of malware linked to the infamous cyber espionage group, Salt Typhoon. They've been targeting critical infrastructure, and it's up to you to uncover their tactics and techniques.


Herramientas utilizadas

PowerShell (pwsh en Kali)

Editor de texto (nano, VSCode)

Decodificación Base64

SalineBreeze-2.zip
3 MB
hacktheblue

Herramienta para ejecutar el codigo
```
decrypt.ps1
```

```
param(
    [string]$key
)

# Load original encrypted script
. ./onedrived.ps1 # This only loads variables, not executed because we override Invoke

# Recompute AES objects manually
$k = $key.PadRight(32,'0')
$aes = New-Object System.Security.Cryptography.AesManaged
$aes.KeySize = 256
$aes.Key = [System.Text.Encoding]::UTF8.GetBytes($k)
$aes.IV  = @(0)*16

$bytes = [Convert]::FromBase64String($s)
$plain = $aes.CreateDecryptor().TransformFinalBlock($bytes,0,$bytes.Length)

# Print plaintext
[System.Text.Encoding]::UTF8.GetString($plain)
```




Task 1

What is the name of the malware family associated with the provided file?
Pasa el archivo a virus total, agarra el hash del archivo y busca la familia
igualmente se puede hacer manual:

└─# sha256sum onedrived.ps1 
2fd4a49338d79f4caee4a60024bcd5ecb5008f1d5219263655ef49c54d9acdec  onedrived.ps1
                                                                                   
```Demodex```

Task 2

What .NET cryptographic class is used to perform decryption in the script?
Codigo para desencriptar el powershell:
Dentro del archivo .ps1 esta

>[string] $k=$args[0].ToString().padright(32,'0');$o=New-Object([System.Text.Encoding]::UTF8.GetString( [System.Convert]::FromBase64String("U3lzdGVtLlNlY3VyaXR5LkNyeXB0b2dyYXBoeS5BZXNNYW5hZ2VkCg==")))

Se decodifica el string de base64 y esta la respuesta

```System.Security.Cryptography.AesManaged```

Task 3

The key to decrypt the script must be entered on the command line when running the ps1. What variable holds the key?

```$k```

Task 4

What is the key required to decrypt the base64 encoded data?

Investigando se puede encontrar la respuesta o usando un diccionario como rockyou

```password@123```

Task 5

After decrypting the initial payload, a new PowerShell script assigns a value to the variable ***$cregvalue***. What is that value?

Buscando en el codigo sale la variable ***$cregvalue*** ahi esta la respuesta

```midihelp```

Task 6

The variable $cregdata is associated with binary registry data stored as a base64 blob. What is the SHA-256 hash of the binary data?

Ahora para esto se crea un archivo:
```
hash.py
```
```
                         
import base64, hashlib

cregdata = "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"

binary = base64.b64decode(cregdata)
print("SHA-256:", hashlib.sha256(binary).hexdigest())

```


```3b1c251b0b37b57b755d4545a7dbbe4c29c15baeca4fc2841f82bc80ea877b66```

Task 7

What is the name of the malicious service?

```$svcname = 'MsMp4Hw'```

Task 8

What is the full path of the malicious DLL?

```
$svcdllpath = 'C:\Windows\System32\msmp4dec.dll'
```

Task 9

What registry path is used to associate the service group (msgroup) with the malicious service, so that it can be launched by svchost.exe?

```
$ret = New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SvcHost" -Name $svcgroup -Value 
$svcname -PropertyType MultiString -Force; if ( $ret -eq $null ) { Write-Output "E:create svc reg1";exit; }
```

Task 10

What is the line of code responsible for starting the malicious service after it has been installed?

```Start-Service -name $svcname;```

## Ahora para las ultimas 3 tenemos que crear el DLL con los datos presentes:

1. como el codigo de svchostdata es muy grande vamos a ponerlo en un archivo
```
cat salida.txt | grep "svchostdata =" > svc2.txt
```
2. Limpiamos el archivo para que solo sea base64, por lo que se debe quitar esto
```
$svchostdata = '
';
try{$svchostbin=$([System.Convert]::FromBase64String($svchostdata));}catch{ Write-Output "E0" ; exit;};
```
Y solo dejar el valor base64

3. Crea un archivo dll.b64 y pega el contenido de $svchostdata dentro:
```
nano dll.b64
```
4. Decodifica:
```
cat dll.b64 | base64 -d > msmp4dec.dll
```

5. Ya tienes tu DLL:
```
file msmp4dec.dll
```

6. Desensámblalo:
```
objdump -d msmp4dec.dll | grep -A3 -i sleep
```
o 
```
strings msmp4dec.dll | grep Sleep
```

>Ahora se puede seguir con las demas preguntas

Task 11

Which Windows API does the DLL import to obtain the local computer name?

```GetComputerNameA```

Task 12

In service DLLs, which function is typically responsible for being called by the Service Control Manager to initiate the service?

```ServiceMain```

Task 13

Inside the previously named function, there are two calls to Sleep: one uses a variable argument and one uses a hard-coded delay. What is the value of the hard-coded delay (in milliseconds)?

```10000```
