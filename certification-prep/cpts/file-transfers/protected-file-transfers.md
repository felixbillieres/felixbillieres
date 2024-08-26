# 🍖 Protected File Transfers

### File Encryption on Windows

One of the simplest methods is the [Invoke-AESEncryption.ps1](https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions/Invoke-AESEncryption.ps1) PowerShell script for enryption of files and strings.

```powershell-session
.EXAMPLE
Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Text "Secret Text" 

Description
-----------
Encrypts the string "Secret Test" and outputs a Base64 encoded ciphertext.
 
.EXAMPLE
Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Text "LtxcRelxrDLrDB9rBD6JrfX/czKjZ2CUJkrg++kAMfs="
 
Description
-----------
Decrypts the Base64 encoded string "LtxcRelxrDLrDB9rBD6JrfX/czKjZ2CUJkrg++kAMfs=" and outputs plain text.
```

Or for a file ->

```
#Encrypt
Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Path file.bin

#Decrypt
Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Path file.bin.aes
```

***

After the script has been transferred, it only needs to be imported as a module ->

```powershell-session
PS C:\htb> Import-Module .\Invoke-AESEncryption.ps1
```

### File Encryption on Linux

To encrypt a file using `openssl` we can select different ciphers like `-aes256.` We can also override the default iterations counts with the option `-iter 100000` and add the option `-pbkdf2` to use the Password-Based Key Derivation Function 2 algorithm

Let's try encrypting /etc/passwd ->

```shell-session
ElFelixi0@htb[/htb]$ openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```

and to decrypt it ->

```shell-session
ElFelixi0@htb[/htb]$ openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd                    
```
