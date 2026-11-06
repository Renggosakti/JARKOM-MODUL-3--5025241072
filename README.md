# Praktikum Modul 3 - Reliable Data Transfer TCP & Static Routing

| Name | NRP | Kelas |
| --- | --- | --- |
| Arya Rangga Putra Pratama | 5025241072 | Jarkom A |

Cakupan: konfigurasi topologi dari `template_modul3.gns3project`, lalu Skenario 3 (loss 1%, latency 100 ms, rate 2 Mbps) untuk `tes1000.bin`, `tes10000.bin`, dan `tes5M.bin` dengan congestion control Reno dan Cubic.

IP prefix: `10.4` (ganti jika prefix dari Linktree berbeda).

> Semua `...` dan screenshot `img/...` diisi dengan hasil aktual dari GNS3 dan Wireshark, bukan estimasi.

---

# Soal 1 - Konfigurasi Topologi dari Template

## 1.1 Import template

1. GNS3: **File > Import portable project** > pilih `template_modul3.gns3project`.
2. Pastikan semua node **mati** (Stop all) sebelum mengisi config.

| Link | Subnet |
| --- | --- |
| NAT1 - RhodesIsland (eth0) | DHCP dari NAT |
| RhodesIsland (eth1) - Lungmen (eth0) | `10.4.1.0/24` |
| RhodesIsland (eth2) - Ursus (eth0) | `10.4.2.0/24` |
| Lungmen (eth1) - Switch1 (e0) | `10.4.3.0/24` |
| Switch1 - Columbia (eth0), Victoria (eth0) | `10.4.3.0/24` |
| Ursus (eth1) - Hub1 (e0) | `10.4.4.0/24` |
| Hub1 - Laterano (eth0), Yan (eth0) | `10.4.4.0/24` |

![Topologi hasil import](img/p1-topologi.png)

## 1.2 Pemetaan IP

| Node | Peran | IP | Gateway |
| --- | --- | --- | --- |
| RhodesIsland | Router ke NAT, titik gangguan `tc` | eth0 DHCP, eth1 `10.4.1.1/24`, eth2 `10.4.2.1/24` | - |
| Lungmen | Router | eth0 `10.4.1.2/24`, eth1 `10.4.3.1/24` | `10.4.1.1` |
| Ursus | Router | eth0 `10.4.2.2/24`, eth1 `10.4.4.1/24` | `10.4.2.1` |
| Victoria | DHCP server `10.4.3.0/24` | eth0 `10.4.3.2/24` | `10.4.3.1` |
| Yan | DHCP server `10.4.4.0/24` | eth0 `10.4.4.2/24` | `10.4.4.1` |
| Laterano | Web server, capture sisi sender | DHCP, reservasi `10.4.4.15` | dari DHCP |
| Columbia | Client uji, capture sisi receiver | DHCP dari Victoria | dari DHCP |

## 1.3 Isi network config (node masih mati)

Klik kanan node > **Edit config** (`/etc/network/interfaces`). Ganti seluruh `[IP Prefix]` dan `[...]`. Gunakan **tab** untuk indentasi.

Catatan urutan baris `up` pada router: `ip_forward` dan `ip route` ditaruh **sebelum** `iptables`, supaya rute tetap terpasang kalau baris iptables gagal.

**RhodesIsland**

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.4.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.4.2.1
	netmask 255.255.255.0

up sysctl -w net.ipv4.ip_forward=1
up ip route add 10.4.3.0/24 via 10.4.1.2 dev eth1
up ip route add 10.4.4.0/24 via 10.4.2.2 dev eth2
up iptables -t nat -A POSTROUTING -j MASQUERADE -s 10.4.0.0/16 -o eth0
```

**Lungmen**

```
auto eth0
iface eth0 inet static
	address 10.4.1.2
	netmask 255.255.255.0
	gateway 10.4.1.1

auto eth1
iface eth1 inet static
	address 10.4.3.1
	netmask 255.255.255.0

up sysctl -w net.ipv4.ip_forward=1
up ip route add 10.4.2.0/24 via 10.4.1.1 dev eth0
up ip route add 10.4.4.0/24 via 10.4.1.1 dev eth0
```

**Ursus**

```
auto eth0
iface eth0 inet static
	address 10.4.2.2
	netmask 255.255.255.0
	gateway 10.4.2.1

auto eth1
iface eth1 inet static
	address 10.4.4.1
	netmask 255.255.255.0

up sysctl -w net.ipv4.ip_forward=1
up ip route add 10.4.1.0/24 via 10.4.2.1 dev eth0
up ip route add 10.4.3.0/24 via 10.4.2.1 dev eth0
```

**Victoria**

```
auto eth0
iface eth0 inet static
	address 10.4.3.2
	netmask 255.255.255.0
	gateway 10.4.3.1
	up echo nameserver 8.8.8.8 > /etc/resolv.conf
```

**Yan**

```
auto eth0
iface eth0 inet static
	address 10.4.4.2
	netmask 255.255.255.0
	gateway 10.4.4.1
	up echo nameserver 8.8.8.8 > /etc/resolv.conf
```

**Laterano dan Columbia**: bawaan template, tidak diubah.

```
auto eth0
iface eth0 inet dhcp
```

![Edit config tiap node](img/p1-edit-config.png)

## 1.4 Start node dan verifikasi routing + internet

**Start all.** Config hanya terbaca saat node start; kalau config diubah setelah node jalan, **stop lalu start ulang** node tersebut.

Verifikasi berurutan dari router ke ujung.

**1. RhodesIsland**

```
ip -br addr
ip route show
cat /proc/sys/net/ipv4/ip_forward
iptables -t nat -L -n -v
ping -c 3 google.com
```

Hasil yang benar:
- eth0 dapat IP dari NAT, eth1 `10.4.1.1/24`, eth2 `10.4.2.1/24`.
- Ada rute `10.4.3.0/24 via 10.4.1.2` dan `10.4.4.0/24 via 10.4.2.2`.
- `ip_forward` = `1`.
- Aturan MASQUERADE `10.4.0.0/16 -o eth0` ada.

Jika `iptables: not found`: `apk update && apk add iptables`, lalu jalankan manual `iptables -t nat -A POSTROUTING -j MASQUERADE -s 10.4.0.0/16 -o eth0`.
Jika `ping google.com` gagal karena DNS: `echo "nameserver 8.8.8.8" > /etc/resolv.conf`.

**2. Lungmen dan Ursus**

```
ip -br addr
ip route show
cat /proc/sys/net/ipv4/ip_forward
```

| Router | Tujuan | Next hop |
| --- | --- | --- |
| Lungmen | `10.4.2.0/24`, `10.4.4.0/24`, default | `10.4.1.1` (eth0) |
| Lungmen | `10.4.1.0/24`, `10.4.3.0/24` | langsung (eth0, eth1) |
| Ursus | `10.4.1.0/24`, `10.4.3.0/24`, default | `10.4.2.1` (eth0) |
| Ursus | `10.4.2.0/24`, `10.4.4.0/24` | langsung (eth0, eth1) |

**3. Victoria dan Yan**

```
ip -br addr
ip route
ping -c 3 google.com
```

`ip route` **wajib** memuat `default via 10.4.3.1 dev eth0` (Victoria) atau `default via 10.4.4.1 dev eth0` (Yan).

Ping silang:

```
# dari Victoria
ping -c 3 10.4.4.2

# dari Yan
ping -c 3 10.4.3.2
```

**Troubleshooting `Network is unreachable` di Victoria/Yan.** Artinya tidak ada rute (default route belum terpasang). Cek `ip -br addr`, `ip route`, dan `cat /etc/network/interfaces`:
- Jika `default` tidak ada, tes cepat: `ip route add default via 10.4.3.1 dev eth0` (Victoria) / `ip route add default via 10.4.4.1 dev eth0` (Yan).
- Jika eth0 tanpa IP, file interfaces tidak terbaca: cari `[IP Prefix]` yang belum diganti atau typo, lalu stop/start ulang node.
- Jika baris `up echo ...` dicurigai menggagalkan `ifup`, hapus baris itu, start ulang, lalu isi `resolv.conf` manual.
- Jika ping antar subnet timeout (bukan unreachable): cek `ip_forward` dan tabel rute di ketiga router.

![Verifikasi router](img/p1-verifikasi-router.png)

![Ping Victoria - Yan](img/p1-ping-victoria-yan.png)

## 1.5 DHCP server kea-dhcp4 (Victoria dan Yan)

Pasang paket (butuh internet dari langkah 1.4), di Victoria dan Yan:

```
bash /root/install_pkg.sh
apk add tmux
```

Isi skrip: `apk update && apk add vim && apk add kea-dhcp4`.

**Victoria** - `/root/start_dhcp.sh` (ganti `[...]` template):

```
#!/bin/bash

kill $(pidof kea-dhcp4) 2>/dev/null

cat > /etc/kea/kea-dhcp4.conf << 'EOF'
{
  "Dhcp4": {
    "interfaces-config": { "interfaces": ["eth0"] },
    "lease-database": { "type": "memfile", "persist": true, "name": "/var/lib/kea/kea-leases4.csv" },
    "valid-lifetime": 3600,
    "subnet4": [
      {
        "id": 1,
        "subnet": "10.4.3.0/24",
        "pools": [ { "pool": "10.4.3.100 - 10.4.3.120" } ],
        "option-data": [
          { "name": "routers", "data": "10.4.3.1" },
          { "name": "domain-name-servers", "data": "8.8.8.8" }
        ]
      }
    ]
  }
}
EOF

mkdir -p /var/lib/kea /run/kea
kea-dhcp4 -t /etc/kea/kea-dhcp4.conf

kea-dhcp4 -d -c /etc/kea/kea-dhcp4.conf
```

**Yan** - `/root/dhcp_start.sh`. Template Yan kehilangan tanda `[` pembuka pada `"option-data"`; sudah ditambahkan di bawah.

MAC Laterano harus dibaca **dari project ini** (bisa berbeda tiap import). Di console Laterano:

```
ip link show eth0
```

Salin nilai `link/ether` ke `hw-address`:

```
#!/bin/bash

kill $(pidof kea-dhcp4) 2>/dev/null

cat > /etc/kea/kea-dhcp4.conf << 'EOF'
{
"Dhcp4": {
    "interfaces-config": {
        "interfaces": [ "eth0" ]
    },
    "lease-database": {
        "type": "memfile",
        "persist": true,
        "name": "/var/lib/kea/kea-leases4.csv"
    },
    "valid-lifetime": 3600,
    "subnet4": [
        {
            "id": 1,
            "subnet": "10.4.4.0/24",
            "pools": [ { "pool": "10.4.4.50 - 10.4.4.60" } ],
            "option-data": [
                {
                    "name": "routers",
                    "data": "10.4.4.1"
                }, {
                    "name": "domain-name-servers",
                    "data": "8.8.8.8"
                }
            ],
            "reservations": [
                { "02:42:3b:b9:fa:00 brd ff:ff:ff:ff:ff:ff
": "<MAC eth0 Laterano>", "ip-address": "10.4.4.15" }
            ]
        }
    ]
}
}
EOF

mkdir -p /var/lib/kea /run/kea
kea-dhcp4 -t /etc/kea/kea-dhcp4.conf

kea-dhcp4 -d -c /etc/kea/kea-dhcp4.conf
```

Jalankan (foreground, jadi pakai tmux atau console kedua):

```
# Victoria
tmux
bash /root/start_dhcp.sh

# Yan
tmux
bash /root/dhcp_start.sh
```

Harus muncul `Syntax check succeeded` (atau `DHCP4_CONFIG...` tanpa error) lalu log `DHCP4_STARTING`.

![Victoria dan Yan: kea-dhcp4 berjalan](img/p1-kea.png)

## 1.6 Lease client (Laterano dan Columbia)

```
# Laterano (console)
udhcpc -i eth0
ip -br addr
ip route

# Columbia (xterm)
udhcpc -i eth0
ip -br addr
ip route
```

- Laterano harus mendapat `10.4.4.15/24` dari Yan.
- Columbia mendapat alamat `10.4.3.100-10.4.3.120` dari Victoria.

Cek konektivitas:

```
# Columbia
ping -c 4 10.4.4.15

# Laterano
ping -c 3 google.com
```

![Lease Laterano dan Columbia](img/p1-lease-client.png)

---

# Soal 2 - Skenario 3 untuk 3 Berkas x Reno dan Cubic

## 2.a Kondisi jaringan di RhodesIsland (loss 1%, delay 100 ms, rate 2 Mbps pada eth1 dan eth2)

Pada RhodesIsland (sama dengan skrip bawaan `/root/bagian_c.sh`):

```
tc qdisc del dev eth1 root 2>/dev/null
tc qdisc del dev eth2 root 2>/dev/null
tc qdisc add dev eth1 root handle 1: tbf rate 2mbit burst 32kbit latency 100ms
tc qdisc add dev eth1 parent 1:1 handle 10: netem delay 100ms loss 1%
tc qdisc add dev eth2 root handle 1: tbf rate 2mbit burst 32kbit latency 100ms
tc qdisc add dev eth2 parent 1:1 handle 10: netem delay 100ms loss 1%
tc qdisc show dev eth1
tc qdisc show dev eth2
```

Cek efek delay dari Columbia: `ping -c 5 10.4.4.15`. RTT dasar sekitar 200 ms (delay 100 ms di arah data dan arah ACK). Rate 2 Mbps = 250.000 B/s.

![tc qdisc show eth1 dan eth2](img/p2-tc-qdisc.png)

## 2.b Persiapan Laterano: python3, berkas uji, HTTP server (2.c soal)

Di Laterano:

```
bash /root/install_pkg.sh      # atau: apk update && apk add python3
apk add tmux
```

Pastikan `python3 --version` jalan. Buat berkas uji pada **direktori yang sama dengan direktori server** (`/root/myscripts/myweb`):

```
mkdir -p /root/myscripts/myweb
cd /root/myscripts/myweb
dd if=/dev/urandom of=tes1000.bin bs=1 count=1000
dd if=/dev/urandom of=tes10000.bin bs=10 count=1000
dd if=/dev/urandom of=tes5M.bin bs=5000 count=1000
ls -l
```

| Berkas | Ukuran |
| --- | --- |
| `tes1000.bin` | 1.000 B |
| `tes10000.bin` | 10.000 B |
| `tes5M.bin` | 5.000.000 B |

Jalankan HTTP server di tmux (`start_http.sh` sudah ada di template):

```
tmux
cd /root/myscripts
./start_http.sh
```

Biarkan berjalan. Gunakan panel tmux lain (`Ctrl+b` lalu `%`, pindah panel dengan `Ctrl+b` + panah) atau console kedua untuk perintah Laterano berikutnya.

Verifikasi dari Columbia:

```
curl -I http://10.4.4.15:9999/tes1000.bin
```

![Laterano: ls -l dan HTTP server](img/p2-http-server.png)

## 2.c Fungsi bantu di Columbia

Di xterm Columbia, definisikan satu kali:

```
T() { curl -o /dev/null -s -w "size=%{size_download}B speed=%{speed_download}B/s total=%{time_total}s\n" http://10.4.4.15:9999/$1; }
```

Pemakaian: `T tes1000.bin`.

## 2.d Prosedur satu kasus uji (diulang 6 kali)

Urutan kasus sesuai soal: **Cubic** (`tes1000`, `tes10000`, `tes5M`), lalu **Reno** (`tes1000`, `tes10000`, `tes5M`).

1. **Laterano**: pastikan algoritma benar dan bersihkan metrics.

   ```
   ip tcp_metrics flush
   sysctl net.ipv4.tcp_congestion_control
   ```

2. **Mulai capture** (filter tampilan `tcp.port == 9999`), keduanya:
   - link Hub1 - Laterano (eth0 Laterano): klik kanan link > **Start capture**
   - link Switch1 - Columbia (eth0 Columbia): klik kanan link > **Start capture**
3. **Columbia**: jalankan satu download dan catat satu baris output.

   ```
   T tes5M.bin
   ```

4. **Stop capture** setelah curl selesai. Simpan dengan nama `<algoritma>-<berkas>-<node>.pcapng`, contoh `cubic-tes5M-laterano.pcapng`, `cubic-tes5M-columbia.pcapng`.

## 2.e Pengujian Cubic (Laterano sebagai sender)

Di Laterano:

```
sysctl -w net.ipv4.tcp_no_metrics_save=1
sysctl -w net.ipv4.tcp_congestion_control=cubic
ip tcp_metrics flush
sysctl net.ipv4.tcp_congestion_control
```

Output harus `net.ipv4.tcp_congestion_control = cubic`. Lalu jalankan prosedur 2.d untuk `tes1000.bin`, `tes10000.bin`, `tes5M.bin`.

| Kasus | Output curl | Capture |
| --- | --- | --- |
| Cubic, tes1000.bin | `size=... speed=... total=...` | `cubic-tes1000-laterano.pcapng`, `cubic-tes1000-columbia.pcapng` |
| Cubic, tes10000.bin | `size=... speed=... total=...` | `cubic-tes10000-laterano.pcapng`, `cubic-tes10000-columbia.pcapng` |
| Cubic, tes5M.bin | `size=... speed=... total=...` | `cubic-tes5M-laterano.pcapng`, `cubic-tes5M-columbia.pcapng` |

![Cubic: output curl](img/p2-curl-cubic.png)

## 2.f Pengujian Reno (kembali ke Laterano)

Di Laterano:

```
sysctl -w net.ipv4.tcp_no_metrics_save=1
sysctl -w net.ipv4.tcp_congestion_control=reno
ip tcp_metrics flush
sysctl net.ipv4.tcp_congestion_control
```

Output harus `net.ipv4.tcp_congestion_control = reno`. Jika `reno` ditolak, cek `cat /proc/sys/net/ipv4/tcp_available_congestion_control`. Lalu ulangi prosedur 2.d untuk ketiga berkas.

| Kasus | Output curl | Capture |
| --- | --- | --- |
| Reno, tes1000.bin | `size=... speed=... total=...` | `reno-tes1000-laterano.pcapng`, `reno-tes1000-columbia.pcapng` |
| Reno, tes10000.bin | `size=... speed=... total=...` | `reno-tes10000-laterano.pcapng`, `reno-tes10000-columbia.pcapng` |
| Reno, tes5M.bin | `size=... speed=... total=...` | `reno-tes5M-laterano.pcapng`, `reno-tes5M-columbia.pcapng` |

![Reno: output curl](img/p2-curl-reno.png)

---

# Soal 3 dan 4 - Tabel Spreadsheet dari Data Aktual

Sumber tiap kolom:

| Kolom | Sumber | Cara |
| --- | --- | --- |
| RTT Rata-rata (ms) | capture **Laterano** | rata-rata `tcp.analysis.ack_rtt` pada ACK yang diterima Laterano (filter di bawah) |
| Retransmisi | capture **Laterano** | jumlah paket pada filter `tcp.analysis.retransmission` |
| Throughput (B/s) | output `curl` | `speed_download` |
| Waktu Transfer (s) | output `curl` | `time_total` |
| Pola cwnd | Time - Sequence (Stevens) | bentuk kurva (Soal 5) |

Dengan `tshark` (di komputer host, pada file `.pcapng` hasil capture), contoh untuk `cubic-tes5M-laterano.pcapng`:

```
# RTT rata-rata (ms): ACK yang masuk ke Laterano (port tujuan 9999)
tshark -r cubic-tes5M-laterano.pcapng -Y "tcp.analysis.ack_rtt && tcp.dstport==9999" -T fields -e tcp.analysis.ack_rtt | awk '{s+=$1; n++} END {printf "%.2f ms\n", s/n*1000}'

# jumlah retransmisi
tshark -r cubic-tes5M-laterano.pcapng -Y "tcp.analysis.retransmission" | wc -l
```

Lewat Wireshark: pasang filter, baca `Displayed: N` di status bar untuk retransmisi. Untuk RTT, jadikan `tcp.analysis.ack_rtt` sebagai kolom (Apply as Column), **File > Export Packet Dissections > As CSV**, lalu rata-ratakan dengan `AVERAGE` di spreadsheet. Gunakan filter yang **sama** untuk keenam kasus.

![Wireshark: filter retransmission dan ack_rtt di capture Laterano](img/p3-filter-wireshark.png)

## Tabel hasil

| Berkas | Algoritma | RTT Rata-rata (ms) | Retransmisi | Throughput (B/s) | Waktu Transfer (s) | Pola cwnd |
| --- | --- | --- | --- | --- | --- | --- |
| tes1000.bin | Reno | ... | ... | ... | ... | ... |
| tes1000.bin | Cubic | ... | ... | ... | ... | ... |
| tes10000.bin | Reno | ... | ... | ... | ... | ... |
| tes10000.bin | Cubic | ... | ... | ... | ... | ... |
| tes5M.bin | Reno | ... | ... | ... | ... | ... |
| tes5M.bin | Cubic | ... | ... | ... | ... | ... |

Tabel yang sama dibuat di Google Sheets atau Excel untuk didemokan ke asisten.

![Spreadsheet hasil pengujian](img/p3-spreadsheet.png)

---

# Soal 5 - Pola cwnd pada Time - Sequence (Stevens)

## Cara membaca grafik

1. Buka capture **Laterano**. Pilih satu paket data **dari Laterano ke Columbia** (source port `9999`).
2. **Statistics > TCP Stream Graphs > Time Sequence (Stevens)**.
3. Grafik ini menampilkan sequence number terhadap waktu, **bukan** nilai cwnd langsung. cwnd dibaca dari perubahan kemiringan kurva.

| Pola | Ciri pada grafik |
| --- | --- |
| Slow start saja | naik cepat, selesai dalam 1-2 RTT, tanpa penurunan |
| Sawtooth curam (khas Reno) | naik linear, kemiringan turun tajam di tiap loss, berulang |
| Lengkung dengan plateau (khas Cubic) | setelah loss naik cepat, melandai dekat titik sebelum loss, lalu naik cepat lagi |

## Pola yang teramati (isi sesuai grafik aktual)

| Berkas | Reno | Cubic |
| --- | --- | --- |
| tes1000.bin | `...` | `...` |
| tes10000.bin | `...` | `...` |
| tes5M.bin | `...` | `...` |

![Stevens tes5M Reno](img/p5-stevens-reno-5M.png)

![Stevens tes5M Cubic](img/p5-stevens-cubic-5M.png)

---

# Soal 6 - Perbandingan Hasil

## 6.1 Antar ukuran berkas (algoritma sama)

- **Reno**: throughput `tes1000` `...` B/s, `tes10000` `...` B/s, `tes5M` `...` B/s. Waktu transfer `...` s, `...` s, `...` s. Retransmisi `...`, `...`, `...`.
- **Cubic**: throughput `tes1000` `...` B/s, `tes10000` `...` B/s, `tes5M` `...` B/s. Waktu transfer `...` s, `...` s, `...` s. Retransmisi `...`, `...`, `...`.

Analisis: tulis dari data di atas. Hal yang perlu dibahas: throughput berkas kecil jauh di bawah batas 250.000 B/s karena waktu didominasi handshake dan slow start, sedangkan `tes5M.bin` mendekati kapasitas link karena transfer cukup lama untuk cwnd tumbuh sampai dibatasi rate 2 Mbps.

## 6.2 Antar algoritma (ukuran berkas sama)

| Berkas | Selisih throughput (Cubic - Reno) | Selisih waktu (Cubic - Reno) | Selisih retransmisi (Cubic - Reno) |
| --- | --- | --- | --- |
| tes1000.bin | `...` B/s | `...` s | `...` |
| tes10000.bin | `...` B/s | `...` s | `...` |
| tes5M.bin | `...` B/s | `...` s | `...` |

---

# Soal 7 - Mengapa Perbedaan Algoritma Lebih Terlihat pada tes5M.bin

1. **Initial window**: Linux memulai dengan cwnd 10 segmen (sekitar 14,5 KB pada MSS 1448 B). `tes1000.bin` (1 segmen) dan `tes10000.bin` (sekitar 7 segmen) muat dalam satu initial window, selesai dalam 1-2 RTT tanpa masuk congestion avoidance.
2. **Reno dan Cubic identik sebelum ada loss**: keduanya memakai slow start yang sama. Perbedaan baru muncul di congestion avoidance dan pemulihan setelah loss (Reno: cwnd dibagi dua lalu naik 1 segmen per RTT; Cubic: cwnd turun ke 70% lalu naik mengikuti fungsi kubik terhadap waktu sejak loss terakhir).
3. **Kemungkinan loss**: dengan loss acak 1%, berkas kecil hanya mengirim beberapa paket sehingga umumnya tidak mengalami loss (retransmisi aktual: `...`). `tes5M.bin` sekitar 3.450 segmen, sehingga loss dan pemulihan terjadi berulang kali (retransmisi aktual: `...`).
4. **Durasi transfer**: `tes5M.bin` berlangsung `...` s pada RTT sekitar 200 ms, cukup untuk ratusan RTT, sehingga perbedaan laju pertumbuhan cwnd terakumulasi pada throughput dan waktu transfer. Pada berkas kecil, selisih hanya variasi acak.

---

# Kesimpulan

1. **Kondisi Cubic unggul dibanding Reno**: pada transfer panjang dengan RTT tinggi dan loss berulang. Setelah loss Cubic hanya menurunkan cwnd sekitar 30% dan menumbuhkannya sebagai fungsi waktu sejak loss terakhir (tidak bergantung RTT), sedangkan Reno menurunkan 50% dan hanya menambah 1 segmen per RTT, sehingga pada RTT sekitar 200 ms pemulihannya lambat. Hasil aktual `tes5M.bin`: throughput Cubic `...` B/s vs Reno `...` B/s, waktu `...` s vs `...` s, retransmisi `...` vs `...`; pola kurva Reno `...`, Cubic `...`. *(Jika hasil aktual justru Reno lebih baik, tulis itu apa adanya dan jelaskan: BDP kecil sekitar 50 KB, loss acak, dan satu kali percobaan per kasus membuat selisih bisa berasal dari variasi loss.)*
2. **Pengaruh ukuran berkas**: berkas kecil selesai di initial window dengan sedikit atau tanpa loss, sehingga Reno dan Cubic berperilaku sama dan throughput dibatasi handshake serta slow start. Berkas besar mengalami banyak loss dan pemulihan, sehingga p
