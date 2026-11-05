| Name           | NRP        | Kelas     |
| ---            | ---        | ----------|
| Arya Rangga Putra Pratama | 5025241072 | Jarkom A |

> [!IMPORTANT]
> Remember to use the IP prefix allocation provided earlier for this pre-lab assignment (pra-praktikan) and for future ones.
> Access it on [Linktree](https://linktr.ee/jarkom)

> [!TIP]
> **Due Date : `5th November 2026, 23.00 WIB`**

#### Put your GNS3 Project files here (gdrive or github upload)!

`Put file URL here`

---

# Bagian A - Reliable Data Transfer pada Kondisi Normal

## A.3 - Persiapan Topologi
 
> Langkah kerja, topologi, dan persiapan konfigurasi network (Pre-Praktikum). Seluruh pengujian pada Bagian A - D menggunakan topologi yang sama.
>
> *Work steps, topology, and network configuration preparation (Pre-Practicum). All tests in Parts A - D use the same topology.*
 
#### Soal A.3.1 - Topologi GNS3
 
> Buat topologi sesuai gambar dan tabel pemetaan node.
>
> *Build the topology according to the figure and the node mapping table.*
 
**Answer:**

Topologi dibuat dengan IP prefix `10.4`. Columbia dan Victoria terhubung ke Switch1, lalu ke Lungmen (eth1). Lungmen (eth0) terhubung ke RhodesIsland (eth1), dan RhodesIsland (eth0) terhubung ke NAT1. RhodesIsland (eth2) terhubung ke Ursus (eth0), dan Ursus (eth1) terhubung ke Hub1 yang menghubungkan Laterano dan Yan.

| Node | Appliance | IP Address | Gateway |
| --- | --- | --- | --- |
| RhodesIsland | netics-pc/netics-server | eth0: DHCP (NAT1), eth1: `10.4.1.1/24`, eth2: `10.4.2.1/24` | - |
| Lungmen | netics-pc/netics-server | eth0: `10.4.1.2/24`, eth1: `10.4.3.1/24` | `10.4.1.1` |
| Ursus | netics-pc/netics-server | eth0: `10.4.2.2/24`, eth1: `10.4.4.1/24` | `10.4.2.1` |
| Victoria | netics-pc/netics-server | eth0: `10.4.3.2/24` (statis), DHCP server pool `10.4.3.100-10.4.3.120` | `10.4.3.1` |
| Yan | netics-pc/netics-server | eth0: `10.4.4.2/24` (statis), DHCP server pool `10.4.4.50-10.4.4.60` | `10.4.4.1` |
| Laterano | netics-pc/netics-server | DHCP client, reservasi `10.4.4.15` | dari DHCP |
| Columbia | netics-pc-desktop | DHCP client dari Victoria | dari DHCP |

![Topologi GNS3 lengkap](img/a31-topologi.png)
 
#### Soal A.3.2 - Konfigurasi IP / Network & Static Routing
 
> Tunjukkan konfigurasi `/etc/network/interfaces` tiap router (RhodesIsland, Lungmen, Ursus), command `ip route add` (dapat dikonfigurasikan otomatis melalui `init.sh`), hasil `ip route show` pada ketiga router, serta bukti ping antara Victoria dan Yan.
>
> *Show the `/etc/network/interfaces` config of each router (RhodesIsland, Lungmen, Ursus), the `ip route add` commands (can be automated through `init.sh`), the `ip route show` output on all three routers, and proof of ping between Victoria and Yan.*
 
**Answer:**

**`/etc/network/interfaces` pada router**

RhodesIsland:

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
```

Lungmen:

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
```

Ursus:

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
```

![RhodesIsland: interfaces](img/a32-interfaces-rhodesisland.png)

![Lungmen: interfaces](img/a32-interfaces-lungmen.png)

![Ursus: interfaces](img/a32-interfaces-ursus.png)

**`/etc/network/interfaces` pada Victoria dan Yan**

Victoria:

```
auto eth0
iface eth0 inet static
address 10.4.3.2
netmask 255.255.255.0
gateway 10.4.3.1
```

Yan:

```
auto eth0
iface eth0 inet static
address 10.4.4.2
netmask 255.255.255.0
gateway 10.4.4.1
```

![Victoria dan Yan: interfaces](img/a32-interfaces-victoria-yan.png)

**Static routing melalui `/root/init.sh`**

RhodesIsland:

```
#!/bin/bash

sysctl -w net.ipv4.ip_forward=1
ip route add 10.4.3.0/24 via 10.4.1.2 dev eth1
ip route add 10.4.4.0/24 via 10.4.2.2 dev eth2
```

Lungmen:

```
#!/bin/bash

ip route add 10.4.2.0/24 via 10.4.1.1 dev eth0
ip route add 10.4.4.0/24 via 10.4.1.1 dev eth0
```

Ursus:

```
#!/bin/bash

ip route add 10.4.1.0/24 via 10.4.2.1 dev eth0
ip route add 10.4.3.0/24 via 10.4.2.1 dev eth0
```

Rute yang terbentuk dicek pada setiap router dengan:

```
ip route show
```

| Router | Tujuan | Next hop | Interface |
| --- | --- | --- | --- |
| RhodesIsland | `10.4.1.0/24` | langsung | eth1 |
| RhodesIsland | `10.4.2.0/24` | langsung | eth2 |
| RhodesIsland | `10.4.3.0/24` | `10.4.1.2` (Lungmen) | eth1 |
| RhodesIsland | `10.4.4.0/24` | `10.4.2.2` (Ursus) | eth2 |
| Lungmen | `10.4.1.0/24` | langsung | eth0 |
| Lungmen | `10.4.3.0/24` | langsung | eth1 |
| Lungmen | `10.4.2.0/24`, `10.4.4.0/24` | `10.4.1.1` (RhodesIsland) | eth0 |
| Ursus | `10.4.2.0/24` | langsung | eth0 |
| Ursus | `10.4.4.0/24` | langsung | eth1 |
| Ursus | `10.4.1.0/24`, `10.4.3.0/24` | `10.4.2.1` (RhodesIsland) | eth0 |

![RhodesIsland: ip route show](img/a32-route-rhodesisland.png)

![Lungmen: ip route show](img/a32-route-lungmen.png)

![Ursus: ip route show](img/a32-route-ursus.png)

**Ping Victoria dan Yan**

```
# dari Victoria
ping -c 4 10.4.4.2

# dari Yan
ping -c 4 10.4.3.2
```

Paket melewati tiga router (Lungmen, RhodesIsland, Ursus), sehingga TTL balasan turun dari 64 menjadi 61.

![Victoria ping Yan dan Yan ping Victoria](img/a32-ping-victoria-yan.png)
 
#### Soal A.3.3 - NAT dan DNS
 
> Tunjukkan aturan `iptables` MASQUERADE pada RhodesIsland (`-s [IP Prefix].0.0/16 -o eth0`), `nameserver 8.8.8.8` pada `/etc/resolv.conf` di semua node, dan bukti semua node (RhodesIsland, Lungmen, Ursus, Victoria, Yan) dapat `ping google.com`.
>
> *Show the `iptables` MASQUERADE rule on RhodesIsland (`-s [IP Prefix].0.0/16 -o eth0`), `nameserver 8.8.8.8` in `/etc/resolv.conf` on all nodes, and proof that all nodes (RhodesIsland, Lungmen, Ursus, Victoria, Yan) can `ping google.com`.*
 
**Answer:**

**NAT pada RhodesIsland**

```
apk update
apk add iptables
iptables -t nat -A POSTROUTING -j MASQUERADE -s 10.4.0.0/16 -o eth0
iptables -t nat -L -n -v
```

Baris iptables juga ditulis di `/root/init.sh` RhodesIsland setelah perintah `ip route add` agar aktif otomatis saat node dinyalakan:

```
#!/bin/bash

ip route add 10.4.3.0/24 via 10.4.1.2 dev eth1
ip route add 10.4.4.0/24 via 10.4.2.2 dev eth2
iptables -t nat -A POSTROUTING -j MASQUERADE -s 10.4.0.0/16 -o eth0
```

![RhodesIsland: aturan MASQUERADE](img/a33-iptables.png)

**DNS pada semua node**

Dijalankan pada RhodesIsland, Lungmen, Ursus, Victoria, dan Yan:

```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
cat /etc/resolv.conf
```

**Ping google.com**

```
ping -c 4 google.com
```

![RhodesIsland: resolv.conf dan ping google.com](img/a33-ping-rhodesisland.png)

![Lungmen: resolv.conf dan ping google.com](img/a33-ping-lungmen.png)

![Ursus: resolv.conf dan ping google.com](img/a33-ping-ursus.png)

![Victoria: resolv.conf dan ping google.com](img/a33-ping-victoria.png)

![Yan: resolv.conf dan ping google.com](img/a33-ping-yan.png)
 
#### Soal A.3.4 - DHCP Server (kea-dhcp4) pada Yan dan Victoria
 
> Tunjukkan `/etc/kea/kea-dhcp4.conf` pada Yan (termasuk reservasi Laterano `[IP Prefix].4.15`) dan Victoria menggunakan kea-dhcp4 (bukan udhcpd), bukti `kea-dhcp4` berjalan, serta bukti lease client (`udhcpc -i eth0` dan `ip -br addr`) pada Laterano dan Columbia.
>
> *Show `/etc/kea/kea-dhcp4.conf` on Yan (including the Laterano reservation `[IP Prefix].4.15`) and Victoria using kea-dhcp4 (not udhcpd), proof that `kea-dhcp4` is running, plus client lease evidence (`udhcpc -i eth0` and `ip -br addr`) on Laterano and Columbia.*
 
**Answer:**

**Instalasi kea-dhcp4 (Yan dan Victoria)**

```
apk update
apk add kea-dhcp4
```

**`/etc/kea/kea-dhcp4.conf` pada Victoria** (subnet `10.4.3.0/24`, pool `10.4.3.100-10.4.3.120`, gateway Lungmen `10.4.3.1`):

```
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
```

**MAC address eth0 Laterano** (untuk reservasi) diambil dari Laterano:

```
ip link show eth0
```

**`/etc/kea/kea-dhcp4.conf` pada Yan** (subnet `10.4.4.0/24`, pool `10.4.4.50-10.4.4.60`, gateway Ursus `10.4.4.1`, reservasi Laterano `10.4.4.15`):

```
{
  "Dhcp4": {
    "interfaces-config": { "interfaces": ["eth0"] },
    "lease-database": { "type": "memfile", "persist": true, "name": "/var/lib/kea/kea-leases4.csv" },
    "valid-lifetime": 3600,
    "subnet4": [
      {
        "id": 1,
        "subnet": "10.4.4.0/24",
        "pools": [ { "pool": "10.4.4.50 - 10.4.4.60" } ],
        "option-data": [
          { "name": "routers", "data": "10.4.4.1" },
          { "name": "domain-name-servers", "data": "8.8.8.8" }
        ],
        "reservations": [
          { "hw-address": "<MAC eth0 Laterano>", "ip-address": "10.4.4.15" }
        ]
      }
    ]
  }
}
```

![Victoria: kea-dhcp4.conf](img/a34-kea-victoria.png)

![Yan: kea-dhcp4.conf dengan reservasi Laterano](img/a34-kea-yan.png)

**Menjalankan kea-dhcp4** (Yan dan Victoria):

```
kea-dhcp4 -t /etc/kea/kea-dhcp4.conf
kea-dhcp4 -c /etc/kea/kea-dhcp4.conf &
ps aux | grep '[k]ea-dhcp4'
```

Perintah start juga ditulis di `/root/init.sh` masing-masing node:

```
#!/bin/bash

kea-dhcp4 -c /etc/kea/kea-dhcp4.conf &
```

![Victoria: kea-dhcp4 berjalan](img/a34-running-victoria.png)

![Yan: kea-dhcp4 berjalan](img/a34-running-yan.png)

**Lease client**

Laterano dan Columbia memakai DHCP pada eth0 (`iface eth0 inet dhcp`).

```
# Laterano
udhcpc -i eth0
ip -br addr

# Columbia
udhcpc -i eth0
ip -br addr
```

Laterano memperoleh `10.4.4.15` dari Yan (reservasi), dan Columbia memperoleh alamat dari pool Victoria `10.4.3.100-10.4.3.120`.

![Laterano: udhcpc dan ip -br addr](img/a34-lease-laterano.png)

![Columbia: udhcpc dan ip -br addr](img/a34-lease-columbia.png)
 
#### Soal A.3.5 - HTTP Server pada Laterano
 
> Pastikan Yan dan Laterano terhubung ke internet dan python3 sudah terpasang di Laterano (seperti modul 2). Tunjukkan `/root/myscripts/start_http.sh` yang dijalankan di tmux (split window) serta pembuatan berkas uji dengan `dd` (`tes1000.bin`, `tes10000.bin`, `tes50000.bin`, `tes5M.bin`).
>
> *Make sure Yan and Laterano can access the internet and python3 is installed on Laterano (as in module 2). Show `/root/myscripts/start_http.sh` running in tmux (split window) and the creation of the test files with `dd` (`tes1000.bin`, `tes10000.bin`, `tes50000.bin`, `tes5M.bin`).*
 
**Answer:**

**Koneksi internet dan instalasi**

Yan dan Laterano dicek koneksinya, lalu python3 dan tmux dipasang pada Laterano:

```
# Yan dan Laterano
echo "nameserver 8.8.8.8" > /etc/resolv.conf
ping -c 4 google.com

# Laterano
apk update
apk add python3 tmux
```

![Yan dan Laterano: ping google.com](img/a35-internet.png)

**`/root/myscripts/start_http.sh`**

```
mkdir -p /root/myscripts
chmod +x /root/myscripts/start_http.sh
```

```
#!/bin/sh

mkdir -p myweb
cd myweb

python3 -m http.server 9999 -d /root/myscripts/myweb
```

**tmux (split window)**

```
tmux
# Ctrl+b lalu %  : membagi layar menjadi panel kiri dan kanan
# Ctrl+b lalu tombol panah : berpindah panel
```

Panel kiri menjalankan HTTP server:

```
cd /root/myscripts
./start_http.sh
```

Panel kanan membuat berkas uji pada `/root/myscripts/myweb`:

```
cd /root/myscripts/myweb
dd if=/dev/random of=tes1000.bin bs=1 count=1000
dd if=/dev/random of=tes10000.bin bs=10 count=1000
dd if=/dev/random of=tes50000.bin bs=50 count=1000
dd if=/dev/random of=tes5M.bin bs=5000 count=1000
ls -l
```

| Berkas | bs x count | Ukuran |
| --- | --- | --- |
| `tes1000.bin` | 1 x 1000 | 1.000 B |
| `tes10000.bin` | 10 x 1000 | 10.000 B |
| `tes50000.bin` | 50 x 1000 | 50.000 B |
| `tes5M.bin` | 5000 x 1000 | 5.000.000 B |

![Laterano: tmux dengan start_http.sh (kiri) dan dd (kanan)](img/a35-tmux-dd.png)
 
#### Soal A.3.6 - Verifikasi Awal dari Columbia
 
> Dari Columbia (VNC/xterm), tunjukkan lease DHCP dari Victoria, `ping` ke `[IP Prefix].4.15`, dan Firefox ke `http://[IP Prefix].4.15:9999` yang menampilkan seluruh berkas uji pada listing.
>
> *From Columbia (VNC/xterm), show the DHCP lease from Victoria, `ping` to `[IP Prefix].4.15`, and Firefox at `http://[IP Prefix].4.15:9999` listing all the test files.*
 
**Answer:**

Dari xterm Columbia:

```
udhcpc -i eth0
ip -br addr
ping -c 4 10.4.4.15
```

Lalu Firefox membuka `http://10.4.4.15:9999`. Listing menampilkan `tes1000.bin`, `tes10000.bin`, `tes50000.bin`, dan `tes5M.bin`.

![Columbia: lease DHCP, ping 10.4.4.15, dan Firefox listing berkas](img/a36-verifikasi-columbia.png)

## Problem A.4 - Three-Way Handshake

> [!NOTE]
> Skenario 1: tanpa gangguan `tc/netem` pada eth1 dan eth2 RhodesIsland. Download `tes10000.bin` dari Columbia ke Laterano (port 9999). Capture Wireshark di **Columbia** dan **Laterano** dengan filter `tcp.port == 9999`.
>
> *Scenario 1: no `tc/netem` disruption on eth1 and eth2 of RhodesIsland. Download `tes10000.bin` from Columbia to Laterano (port 9999). Capture with Wireshark on **Columbia** and **Laterano** using the filter `tcp.port == 9999`.*

 
#### Soal A.4.1 - Persiapan & Prediksi
 
> Pastikan tidak ada qdisc gangguan (`tc qdisc show`/`tc qdisc del`). Tuliskan **prediksi** ISN client pada paket SYN sebelum melihat capture.
>
> *Make sure no disruptive qdisc is active. Write your **prediction** of the client's ISN in the SYN packet before looking at the capture.*
 
**Answer:**

Pada RhodesIsland:

```
tc qdisc show dev eth1
tc qdisc show dev eth2
tc qdisc del dev eth1 root
tc qdisc del dev eth2 root
```

Pesan `Cannot delete qdisc with handle of zero` pada `tc qdisc del` wajar karena belum ada qdisc gangguan.

Persiapan capture: Wireshark dijalankan pada link Columbia - Switch1 (eth0 Columbia) dan link Laterano - Hub1 (eth0 Laterano) dengan display filter `tcp.port == 9999`. Download dilakukan dari xterm Columbia:

```
curl -o /dev/null -s -w "size=%{size_download}B speed=%{speed_download}B/s total=%{time_total}s\n" http://10.4.4.15:9999/tes10000.bin
```

Prediksi saya: ISN client pada paket SYN tidak dimulai dari 0, melainkan nilai acak 32-bit (0 sampai 4.294.967.295) yang dibangkitkan kernel untuk setiap koneksi baru.

![tc qdisc show eth1 dan eth2 pada RhodesIsland](img/a41-tc-qdisc.png)

![Columbia: curl tes10000.bin](img/a41-curl-tes10000.png)
 
#### Soal A.4.2 - Tabel Observasi Three-Way Handshake

> [!NOTE]
> Untuk parameter 6-8 gunakan capture **Laterano**; untuk 9-10 gunakan capture **Columbia**.
>
> *For items 6-8, use the **Laterano** capture; for 9-10, use the **Columbia** capture.*
 
> Isi tabel berikut berdasarkan capture Wireshark.
>
> *Fill in the following table based on the Wireshark capture.*
 
| No | Parameter/Aspek | Field Wireshark | Instruksi / *Instruction* | Jawaban & Screenshot |
| --- | --- | --- | --- | --- |
| 1 | IP sumber dan tujuan<br>*Source and destination IP* | `ip.src`, `ip.dst` | Pastikan koneksi terjadi antara client (pengirim request) dan server (penerima request).<br>*Make sure the connection is between the client (request sender) and the server (request receiver).* | Client Columbia: `ip.src` = `...` (lease dari pool Victoria `10.4.3.100-10.4.3.120`), source port `...`<br>Server Laterano: `ip.dst` = `10.4.4.15`, destination port `9999`<br>Alamat sama pada kedua capture karena tidak ada NAT antar subnet internal (MASQUERADE hanya pada eth0 RhodesIsland).<br>![A.4.2-1 ip.src dan ip.dst](img/a42-01.png) |
| 2 | Proses handshake (3 paket awal)<br>*Handshake process (first 3 packets)* | `tcp.flags.syn`, `tcp.flags.ack` | Amati tiga paket pertama: SYN → SYN-ACK → ACK.<br>*Observe the first three packets: SYN → SYN-ACK → ACK.* | 1. SYN: Columbia ke Laterano, `syn=1`, `ack=0` (frame `...`)<br>2. SYN-ACK: Laterano ke Columbia, `syn=1`, `ack=1` (frame `...`)<br>3. ACK: Columbia ke Laterano, `syn=0`, `ack=1` (frame `...`)<br>![A.4.2-2 three-way handshake](img/a42-02.png) |
| 3 | Prediksi vs aktual: sequence number awal client<br>*Predicted vs actual: client's initial sequence number* | `tcp.seq` (SYN) | Bandingkan prediksi sebelum capture dengan nilai aktual, jelaskan mengapa ISN acak, bukan nol.<br>*Compare your prediction before the capture with the actual value, and explain why the ISN is random, not zero.* | Prediksi: acak, bukan 0.<br>ISN client aktual (`tcp.seq_raw` pada SYN): `...`<br>Wireshark menampilkan `tcp.seq = 0` karena relative sequence numbers aktif; nilai asli terlihat pada `tcp.seq_raw`.<br>ISN dibuat acak agar segmen lama atau duplikat dari koneksi sebelumnya dengan pasangan IP:port yang sama tidak diterima sebagai data koneksi baru, dan agar sequence number sulit ditebak pihak lain (mencegah spoofing dan injeksi paket).<br>![A.4.2-3 ISN client](img/a42-03.png) |
| 4 | Sequence number awal server<br>*Server's initial sequence number* | `tcp.seq` (SYN-ACK) | Tunjukkan byte awal dari sisi server.<br>*Show the initial byte on the server side.* | ISN server (`tcp.seq_raw` pada SYN-ACK): `...`<br>`tcp.ack` pada SYN-ACK = ISN client + 1 = `...`<br>Tiap arah memiliki urutan byte sendiri, sehingga ISN server berbeda dari ISN client.<br>![A.4.2-4 ISN server](img/a42-04.png) |
| 5 | ACK setelah handshake<br>*ACK after the handshake* | `tcp.ack` | Tunjukkan bahwa client mengonfirmasi SYN-ACK dari server.<br>*Show that the client acknowledges the server's SYN-ACK.* | Paket ACK ketiga: `tcp.seq_raw` = ISN client + 1 = `...`, `tcp.ack_raw` = ISN server + 1 = `...`<br>SYN memakai satu nomor urut, sehingga client mengonfirmasi SYN-ACK dengan ack = ISN server + 1.<br>![A.4.2-5 ACK setelah handshake](img/a42-05.png) |
| 6 | Verifikasi SEQ segmen berikutnya = SEQ + panjang payload<br>*Verify next segment's SEQ = SEQ + payload length* | `tcp.seq`, `tcp.len` | Pilih satu segmen data di tengah stream (bukan segmen pertama atau terakhir), lalu buktikan secara manual bahwa segmen berikutnya memiliki `tcp.seq` = `tcp.seq` + `tcp.len` segmen ini.<br>*Choose a data segment in the middle of the stream (not the first or last), then manually prove that the next segment has `tcp.seq` = this segment's `tcp.seq` + `tcp.len`.* | Segmen data dari Laterano ke Columbia (pengirim berkas).<br>`tcp.seq` segmen ini: ...<br>`tcp.len` segmen ini: ...<br>`tcp.seq` berikutnya: ...<br>Perhitungan: ... + ... = ...<br>![A.4.2-6 verifikasi SEQ](img/a42-06.png) |
| 7 | Pola ACK penerima<br>*Receiver's ACK pattern* | `tcp.ack` | Tunjukkan bahwa ACK bertambah kumulatif, menandakan data diterima secara berurutan.<br>*Show that ACKs increase cumulatively, indicating that data is received in order.* | ACK dari Columbia naik secara kumulatif: `...`, `...`, `...` (selisih tiap ACK sesuai `tcp.len` segmen yang dikonfirmasi; satu ACK dapat mengonfirmasi dua segmen sekaligus karena delayed ACK).<br>Nilai ACK tidak turun, menandakan data tiba berurutan.<br>![A.4.2-7 pola ACK kumulatif](img/a42-07.png) |
| 8 | RTT rata-rata dan implikasinya terhadap timer<br>*Average RTT and its effect on the timer* | `tcp.analysis.ack_rtt` | Catat RTT pada salah satu ACK (di Laterano), lalu diskusikan mengapa timer yang terlalu pendek maupun terlalu panjang sama-sama bermasalah bagi kinerja transfer.<br>*Record the RTT from one ACK (on Laterano), then discuss why a timer that is too short and one that is too long both cause transfer performance problems.* | RTT pada salah satu ACK (`tcp.analysis.ack_rtt`): `...` ms; rata-rata dari beberapa ACK: `...` ms.<br>Retransmission timer (RTO) diturunkan dari RTT yang terukur (SRTT dan RTTVAR).<br>Timer terlalu pendek: timeout muncul padahal ACK masih di perjalanan, terjadi retransmisi yang sebenarnya tidak perlu, bandwidth terbuang, dan cwnd turun tanpa alasan.<br>Timer terlalu panjang: segmen yang benar-benar hilang baru dikirim ulang setelah menunggu lama, jalur menganggur dan waktu transfer membengkak.<br>![A.4.2-8 tcp.analysis.ack_rtt](img/a42-08.png) |
| 9 | Status retransmisi<br>*Retransmission status* | `tcp.analysis.retransmission` | Semestinya tidak ada retransmisi, menandakan link sehat.<br>*There should be no retransmissions, indicating a healthy link.* | Filter `tcp.analysis.retransmission` pada capture Columbia: `...` paket (0 paket berarti link sehat).<br>![A.4.2-9 filter retransmisi](img/a42-09.png) |
| 10 | Pola penutupan koneksi<br>*Connection closing pattern* | `tcp.flags.fin`, `tcp.flags.ack` | Tunjukkan bahwa TCP menutup koneksi melalui four-way handshake (FIN/ACK).<br>*Show that TCP closes the connection through a four-way handshake (FIN/ACK).* | Urutan penutupan pada capture Columbia: FIN-ACK dari `...` (frame `...`), ACK dari `...` (frame `...`), FIN-ACK dari `...` (frame `...`), ACK dari `...` (frame `...`).<br>Setiap arah ditutup sendiri-sendiri (FIN lalu ACK), sehingga ada empat langkah; ACK dan FIN dari satu sisi dapat digabung dalam satu paket.<br>![A.4.2-10 penutupan koneksi](img/a42-10.png) |

#### A.4.2 - Pertanyaan Analisis / Refleksi

> 1. Apa fungsi utama sequence number dan acknowledgment number dalam TCP?
>    *What are the main functions of sequence numbers and acknowledgment numbers in TCP?*
>
> 2. Mengapa ACK pada TCP bersifat kumulatif, bukan per-paket?
>    *Why are TCP ACKs cumulative rather than per-packet?*
>
> 3. Apa yang akan terjadi jika salah satu segmen data hilang di tengah pengiriman?
>    *What will happen if a data segment is lost in the middle of a transfer?*
>
> 4. Apa bukti dari hasil Wireshark bahwa TCP berhasil melakukan transfer data secara reliable?
>    *What evidence in the Wireshark results shows that TCP successfully transfers data reliably?*

**Answer:**

1. Sequence number memberi nomor pada setiap byte data yang dikirim, sehingga penerima dapat menyusun data sesuai urutan, mendeteksi data yang hilang atau duplikat, dan mengetahui posisi tiap segmen dalam stream. Acknowledgment number berisi nomor byte berikutnya yang diharapkan penerima, sehingga sekaligus menjadi konfirmasi bahwa semua byte sebelum nomor itu sudah diterima. Keduanya menjadi dasar reliable data transfer: pengirim tahu byte mana yang sudah sampai dan mana yang perlu dikirim ulang.

2. ACK kumulatif cukup menyebut byte berurutan terakhir yang diterima, sehingga satu ACK mengonfirmasi banyak segmen sekaligus. Jumlah ACK menjadi lebih sedikit (juga dengan delayed ACK), overhead jaringan lebih kecil, dan implementasi lebih sederhana. ACK kumulatif juga tahan terhadap ACK yang hilang: ACK berikutnya yang lebih besar otomatis mencakup ACK yang hilang tersebut, sehingga pengirim tidak perlu mengirim ulang data yang sebenarnya sudah diterima.

3. Segmen sesudah segmen yang hilang tetap tiba, tetapi tidak berurutan. Penerima menyimpan data tersebut dan mengirim ACK dengan nomor yang sama berulang (duplicate ACK) yang menunjuk ke byte yang belum diterima. Setelah tiga duplicate ACK, pengirim melakukan fast retransmit tanpa menunggu timer. Jika duplicate ACK tidak cukup muncul, segmen dikirim ulang setelah retransmission timer habis (timeout). Aplikasi di sisi penerima baru menerima data setelah byte yang hilang terisi, sehingga data tetap utuh dan berurutan.

4. Buktinya: handshake lengkap (SYN, SYN-ACK, ACK) dan penutupan koneksi dengan FIN/ACK; `tcp.seq` tiap segmen sama dengan `tcp.seq` + `tcp.len` segmen sebelumnya tanpa celah; `tcp.ack` dari penerima naik kumulatif sampai mencakup seluruh byte; tidak ada flag `tcp.analysis.retransmission`; dan `size_download` pada output `curl` sama dengan ukuran berkas (10000 B).

---

# Bagian B - Retransmission & Duplicate ACK

> [!NOTE]
> Skenario 2: packet loss 1% pada eth1 dan eth2 RhodesIsland. Download `tes5M.bin`. Untuk parameter 2 dan 5 gunakan capture **Columbia**; selain itu gunakan capture **Laterano**.
>
> *Scenario 2: 1% packet loss on eth1 and eth2 of RhodesIsland. Download `tes5M.bin`. For items 2 and 5, use the **Columbia** capture; for the others, use the **Laterano** capture.*

## Problem B.1 - B.3
 
#### Soal B.1 - Persiapan Gangguan & Prediksi
 
> Terapkan `tc qdisc add dev eth1/eth2 root netem loss 1%`. Tuliskan prediksi reaksi penerima terhadap loss sebelum melihat capture.
>
> *Apply `tc qdisc add dev eth1/eth2 root netem loss 1%`. Write your prediction of the receiver's reaction to loss before looking at the capture.*
 
**Answer:**

Pada RhodesIsland:

```
tc qdisc del dev eth1 root
tc qdisc del dev eth2 root
tc qdisc add dev eth1 root netem loss 1%
tc qdisc add dev eth2 root netem loss 1%
tc qdisc show dev eth1
tc qdisc show dev eth2
```

Capture Wireshark di Columbia dan Laterano (filter `tcp.port == 9999`), lalu download dari xterm Columbia:

```
curl -o /dev/null -s -w "size=%{size_download}B speed=%{speed_download}B/s total=%{time_total}s\n" http://10.4.4.15:9999/tes5M.bin
```

Prediksi saya: TCP tidak memakai NAK. Saat ada segmen yang hilang, segmen berikutnya yang tiba tidak berurutan sehingga penerima (Columbia) mengirim ACK dengan nomor yang sama berulang (duplicate ACK). Setelah tiga duplicate ACK, pengirim (Laterano) melakukan fast retransmit tanpa menunggu timeout.

![tc qdisc show eth1 dan eth2 (loss 1%)](img/b1-tc-qdisc.png)

![Columbia: curl tes5M.bin dengan loss 1%](img/b1-curl-tes5M.png)
 
#### Soal B.3 - Tabel Observasi Retransmisi & Duplicate ACK
 
> Isi tabel berikut berdasarkan capture Wireshark dan output `curl`.
>
> *Fill in the following table based on the Wireshark capture and the `curl` output.*
 
| No | Parameter/Aspek | Field Wireshark | Instruksi / *Instruction* | Jawaban & Screenshot |
| --- | --- | --- | --- | --- |
| 1 | Persentase loss yang disimulasikan<br>*Simulated loss percentage* | `tc qdisc show` | Tunjukkan nilai setup packet loss (1%) pada `tc netem`.<br>*Show the packet loss setting (1%) in `tc netem`.* | `tc qdisc show dev eth1` dan `dev eth2` menampilkan `netem ... loss 1%` pada kedua interface.<br>![B.3-1 tc qdisc show](img/b3-01.png) |
| 2 | Prediksi vs aktual: reaksi penerima terhadap loss (Columbia)<br>*Predicted vs actual: receiver's response to loss (Columbia)* | `tcp.analysis.duplicate_ack` | Bandingkan prediksi dengan pola dupACK aktual (ACK bernomor sama berulang), bukan NAK.<br>*Compare your prediction with the actual dupACK pattern (repeated ACKs with the same number), not NAK.* | Prediksi: penerima mengirim duplicate ACK, bukan NAK.<br>Aktual (filter `tcp.analysis.duplicate_ack`): ACK bernomor `...` muncul berulang `...` kali (frame `...` sampai `...`).<br>Prediksi sesuai: TCP tidak memiliki NAK, hanya ACK berulang yang menunjuk byte yang belum diterima.<br>![B.3-2 duplicate ACK pada Columbia](img/b3-02.png) |
| 3 | Jumlah retransmisi<br>*Number of retransmissions* | `tcp.analysis.retransmission` | Tunjukkan paket yang dikirim ulang akibat tidak menerima ACK yang diharapkan.<br>*Show the packets sent again because the expected ACK was not received.* | Filter `tcp.analysis.retransmission` pada capture Laterano: `...` paket.<br>Contoh paket retransmisi: frame `...`, `tcp.seq` = `...`.<br>![B.3-3 retransmisi](img/b3-03.png) |
| 4 | Verifikasi SEQ segmen retransmisi = SEQ segmen asli yang hilang<br>*Verify retransmitted segment's SEQ = original lost segment's SEQ* | `tcp.seq`, `tcp.len` | Bandingkan paket `tcp.analysis.retransmission` dengan segmen aslinya, buktikan bahwa retransmisi mengirim ulang byte yang identik, bukan data baru.<br>*Compare a `tcp.analysis.retransmission` packet with its original segment, and prove that the retransmission sends the same bytes again, not new data.* | Segmen asli: seq = ..., len = ...<br>Retransmisi: seq = ..., len = ...<br>Seq dan len sama, sehingga byte yang dikirim ulang identik, bukan data baru.<br>![B.3-4 perbandingan segmen asli dan retransmisi](img/b3-04.png) |
| 5 | Jumlah duplicate ACK (Columbia)<br>*Number of duplicate ACKs (Columbia)* | `tcp.analysis.duplicate_ack` | Tunjukkan bahwa sedikitnya 3 dupACK berturut-turut memicu fast retransmit.<br>*Show that at least 3 consecutive dupACKs trigger fast retransmit.* | Total duplicate ACK pada capture Columbia: `...` paket.<br>Contoh urutan: dupACK #1 (frame `...`), #2 (frame `...`), #3 (frame `...`), disusul retransmisi (frame `...`).<br>![B.3-5 duplicate ACK berturut-turut](img/b3-05.png) |
| 6 | RTT sebelum dan sesudah loss<br>*RTT before and after loss* | `tcp.analysis.ack_rtt` | Tunjukkan bahwa RTT cenderung meningkat karena waktu recovery.<br>*Show that RTT tends to increase because of recovery time.* | RTT sebelum loss: `...` ms; RTT pada ACK yang mengikuti recovery: `...` ms.<br>RTT naik karena ACK untuk segmen yang hilang baru muncul setelah segmen dikirim ulang.<br>![B.3-6 RTT sebelum dan sesudah loss](img/b3-06.png) |
| 7 | Fast retransmit terdeteksi beserta selisih waktunya<br>*Detected fast retransmit and its time difference* | `tcp.analysis.fast_retransmission`, `frame.time_relative` | Ukur selisih waktu antara dupACK ke-3 dan retransmisi yang menyusul, bandingkan dengan potensi timeout.<br>*Measure the time between the third dupACK and the retransmission that follows, and compare it with a possible timeout.* | Jumlah `tcp.analysis.fast_retransmission`: `...`<br>`frame.time_relative` dupACK ke-3: `...` s; retransmisi: `...` s; selisih: `...` ms.<br>Selisih ini hanya sekitar satu RTT, jauh lebih cepat daripada menunggu retransmission timer (RTO minimum Linux 200 ms).<br>![B.3-7 fast retransmit dan selisih waktu](img/b3-07.png) |
| 8 | Timeout retransmission<br>*Timeout retransmission* | `frame.time_relative` | Terjadi bila tidak ada dupACK yang memicu fast retransmit. Jika tidak muncul, jelaskan apa yang diamati beserta alasannya.<br>*Occurs when there are no dupACKs to trigger fast retransmit. If it does not appear, describe what was observed and why.* | Hasil pengamatan: `...`<br>Timeout retransmission hanya terjadi bila dupACK tidak cukup muncul, misalnya segmen yang hilang berada di ujung transfer atau retransmisinya ikut hilang. Pada loss acak 1% dengan aliran data yang panjang, hampir semua loss mendapat dupACK sehingga dipulihkan lewat fast retransmit.<br>![B.3-8 pengamatan timeout](img/b3-08.png) |
| 9 | Throughput rata-rata<br>*Average throughput* | `curl -w` (`speed_download`) | Bandingkan dengan throughput pada Skenario 1 (tanpa loss).<br>*Compare with the throughput in Scenario 1 (without loss).* | `speed_download` (loss 1%, `tes5M.bin`): `...` B/s.<br>Pembanding tanpa loss (`tes5M.bin`, Tabel E.1): `...` B/s.<br>![B.3-9 speed_download](img/b3-09.png) |
| 10 | Lama transfer total<br>*Total transfer duration* | `curl -w` (`time_total`) | Tunjukkan bahwa waktu transfer meningkat dibanding Skenario 1.<br>*Show that the transfer time increases compared with Scenario 1.* | `time_total` (loss 1%): `...` s.<br>Pembanding tanpa loss (`tes5M.bin`, Tabel E.1): `...` s.<br>![B.3-10 time_total](img/b3-10.png) |
| 11 | Pola congestion window (perkiraan)<br>*Congestion window pattern (estimate)* | `tcp.analysis.bytes_in_flight` | Tunjukkan bahwa cwnd menurun setelah loss, indikasi congestion control aktif.<br>*Show that cwnd decreases after loss, indicating that congestion control is active.* | `bytes_in_flight` sebelum loss: `...` B; sesudah loss: `...` B.<br>Nilai ini hanya perkiraan cwnd karena `bytes_in_flight` dibatasi juga oleh receive window. Penurunannya setelah loss menunjukkan congestion control aktif.<br>![B.3-11 bytes_in_flight](img/b3-11.png) |

#### B.3 - Pertanyaan Analisis / Refleksi

> 1. Bagaimana TCP mengetahui bahwa sebuah segmen hilang?
>    *How does TCP know that a segment is lost?*
>
> 2. Jelaskan perbedaan antara timeout retransmission dan fast retransmit.
>    *Explain the difference between timeout retransmission and fast retransmit.*
>
> 3. Mengapa throughput menurun saat terjadi packet loss?
>    *Why does throughput decrease when packet loss occurs?*
>
> 4. Bagaimana Wireshark menunjukkan bahwa TCP tetap reliable meskipun terjadi packet loss?
>    *How does Wireshark show that TCP remains reliable even when packet loss occurs?*

**Answer:**

1. TCP mengetahui segmen hilang dari dua sinyal: (a) duplicate ACK, yaitu penerima mengirim ACK dengan nomor sama berulang karena menerima segmen di luar urutan, dan (b) timeout, yaitu ACK untuk suatu segmen tidak kunjung datang sampai retransmission timer habis. TCP tidak memakai NAK; kehilangan disimpulkan dari ACK yang tidak maju atau tidak datang.

2. Fast retransmit dipicu oleh tiga duplicate ACK berturut-turut, sehingga pengirim mengirim ulang segmen yang hilang tanpa menunggu timer, kira-kira setelah satu RTT. Timeout retransmission terjadi ketika tidak ada ACK sama sekali sampai RTO habis (biasanya karena tidak ada cukup dupACK, misalnya loss di ujung transfer). Timeout lebih lambat dan lebih berat dampaknya: cwnd diturunkan ke 1 MSS dan kembali ke slow start, sedangkan pada fast retransmit cwnd hanya dikurangi (fast recovery).

3. Pertama, segmen yang hilang harus dikirim ulang dan data di belakangnya tertahan sampai byte yang hilang terisi, sehingga ada waktu terbuang. Kedua, TCP menganggap loss sebagai tanda congestion, sehingga cwnd diturunkan dan laju pengiriman berkurang. Ketiga, retransmisi memakai kapasitas link untuk data yang sama, sehingga goodput menurun. Akibatnya waktu transfer total bertambah.

4. Wireshark menandai paket retransmisi (`tcp.analysis.retransmission`) dan duplicate ACK, tetapi urutan byte pada akhirnya lengkap: segmen retransmisi memiliki `tcp.seq` dan `tcp.len` yang sama dengan segmen asli, ACK penerima terus maju kumulatif sampai mencakup seluruh data, koneksi ditutup normal dengan FIN/ACK, dan `size_download` pada `curl` tetap 5.000.000 B, sama dengan ukuran berkas.

---

# Bagian C - TCP Congestion Control (CUBIC vs Reno)

> [!NOTE]
> Skenario 3: loss 1%, delay 100 ms, rate limit 2 Mbps (tbf + netem) pada eth1 dan eth2 RhodesIsland. Algoritma diatur di **Laterano** (sender). Download `tes5M.bin`. Untuk parameter 4, 5, dan 9 gunakan capture **Columbia**; selain itu gunakan capture **Laterano**. Jangan klaim mengukur cwnd secara langsung dari grafik Time-Sequence / `bytes_in_flight`.
>
> *Scenario 3: 1% loss, 100 ms delay, 2 Mbps rate limit (tbf + netem) on eth1 and eth2 of RhodesIsland. The algorithm is set on **Laterano** (sender). Download `tes5M.bin`. For items 4, 5, and 9 use the **Columbia** capture; for the others use the **Laterano** capture. Do not claim to measure cwnd directly from the Time-Sequence graph / `bytes_in_flight`.*

## Problem C.1 - C.3

#### Soal C.1 - Setup Gangguan & Prediksi Kurva cwnd

> Tunjukkan konfigurasi `tc` (tbf + netem). Tuliskan prediksi bentuk kurva cwnd untuk Reno dan CUBIC sebelum melihat grafik.
>
> *Show the `tc` configuration (tbf + netem). Write your predicted cwnd curve shape for Reno and CUBIC before looking at the actual graphs.*

**Answer:**

Pada RhodesIsland:

```
tc qdisc del dev eth1 root
tc qdisc del dev eth2 root
tc qdisc add dev eth1 root handle 1: tbf rate 2mbit burst 32kbit latency 100ms
tc qdisc add dev eth1 parent 1:1 handle 10: netem delay 100ms loss 1%
tc qdisc add dev eth2 root handle 1: tbf rate 2mbit burst 32kbit latency 100ms
tc qdisc add dev eth2 parent 1:1 handle 10: netem delay 100ms loss 1%
tc qdisc show dev eth1
tc qdisc show dev eth2
```

Delay 100 ms dipasang pada eth1 (arah data ke Columbia) dan eth2 (arah ACK ke Laterano), sehingga RTT dasar sekitar 200 ms. Batas rate 2 Mbps setara 250.000 B/s, sehingga `tes5M.bin` (5.000.000 B) membutuhkan minimal sekitar 20 detik.

Prediksi Reno: setelah loss, cwnd turun tajam menjadi setengahnya (multiplicative decrease), lalu naik linear sekitar 1 MSS per RTT (additive increase), membentuk pola gigi gergaji yang landai.

Prediksi CUBIC: setelah loss, cwnd turun (faktor 0,7), lalu naik cepat mendekati titik cwnd sebelum loss, melambat di sekitar titik tersebut, lalu naik cepat lagi untuk mencari kapasitas baru, mengikuti fungsi kubik terhadap waktu sejak loss terakhir.

![tc qdisc show eth1 dan eth2 (tbf + netem)](img/c1-tc-qdisc.png)

#### Soal C.2 - Konfigurasi Algoritma Congestion Control
 
> Atur algoritma di Laterano (sender) lalu jalankan download `tes5M.bin` dari Columbia untuk masing-masing algoritma.
>
> *Set the algorithm on Laterano (sender), then download `tes5M.bin` from Columbia for each algorithm.*
 
| Langkah / *Step* | CUBIC | Reno |
| --- | --- | --- |
| Command di Laterano<br>*Command on Laterano* | `ip tcp_metrics flush`<br>`sysctl -w net.ipv4.tcp_no_metrics_save=1`<br>`sysctl -w net.ipv4.tcp_congestion_control=cubic`<br>`sysctl net.ipv4.tcp_congestion_control` | `sysctl -w net.ipv4.tcp_congestion_control=reno`<br>`ip tcp_metrics flush`<br>`sysctl net.ipv4.tcp_congestion_control` |
| Command di Columbia<br>*Command on Columbia* | `curl ... tes5M.bin` (sama seperti Skenario 1 dan 2 / *same as Scenarios 1 and 2*) | `curl ... tes5M.bin` (sama seperti CUBIC / *same as CUBIC*) |
| Screenshot | ![C.2 CUBIC: sysctl dan curl](img/c2-cubic.png) | ![C.2 Reno: sysctl dan curl](img/c2-reno.png) |

Perintah `curl` pada Columbia untuk kedua algoritma:

```
curl -o /dev/null -s -w "size=%{size_download}B speed=%{speed_download}B/s total=%{time_total}s\n" http://10.4.4.15:9999/tes5M.bin
```

Capture Wireshark di Columbia dan Laterano (filter `tcp.port == 9999`) dibuat terpisah untuk CUBIC dan Reno. Grafik diambil dari menu Statistics, TCP Stream Graphs: Time - Sequence (Stevens), Round Trip Time, dan Throughput.
 
#### Soal C.3 - Tabel Perbandingan CUBIC vs Reno
 
> Isi tabel berikut dengan hasil aktual untuk CUBIC dan Reno (nilai + screenshot), lalu tulis analisis singkat pada kolom catatan.
> Setiap sel kosong untuk CUBIC dan RENO pada tabel di bawah **wajib** berisi **nilai aktual** dan **screenshot** sebagai bukti, kecuali parameter nomor 1.
>
> *Fill in the following table with actual results for CUBIC and Reno (value + screenshot), then write a short analysis in the notes column.*
> *Every cell for CUBIC and RENO column in the table below **must** contain the **actual value** and a **screenshot** as evidence, except for parameter 1.*
 
| No | Parameter/Aspek | Field Wireshark / Command | CUBIC | Reno | Catatan analisis / *Analysis notes* |
| --- | --- | --- | --- | --- | --- |
| 1 | Prediksi vs aktual: bentuk kurva cwnd<br>*Predicted vs actual: cwnd curve shape* | Time - Sequence Graph | Aktual: `...`<br>![C.3-1 Time-Sequence CUBIC](img/c3-01-cubic.png) | Aktual: `...`<br>![C.3-1 Time-Sequence Reno](img/c3-01-reno.png) | Cocokkan dugaan awal dengan kurva aktual; jelaskan bila berbeda.<br>*Match your initial prediction with the actual curve; explain any differences.*<br>Prediksi: Reno gigi gergaji linear, CUBIC naik cepat mendekati titik sebelum loss. Kesesuaian dengan aktual: `...` |
| 2 | RTT rata-rata (ms)<br>*Average RTT (ms)* | `tcp.analysis.ack_rtt` | `...` ms<br>![C.3-2 RTT rata-rata CUBIC](img/c3-02-cubic.png) | `...` ms<br>![C.3-2 RTT rata-rata Reno](img/c3-02-reno.png) | RTT stabil menunjukkan congestion control yang efisien.<br>*A stable RTT indicates efficient congestion control.*<br>RTT dasar sekitar 200 ms (delay 100 ms pada eth1 dan eth2). Selisih di atas nilai itu berasal dari antrean tbf. Analisis: `...` |
| 3 | RTT maksimum (ms)<br>*Maximum RTT (ms)* | `tcp.analysis.ack_rtt` | `...` ms<br>![C.3-3 RTT maksimum CUBIC](img/c3-03-cubic.png) | `...` ms<br>![C.3-3 RTT maksimum Reno](img/c3-03-reno.png) | RTT tinggi mengindikasikan queue buildup pada jalur.<br>*A high RTT indicates queue buildup along the path.*<br>Analisis: `...` |
| 4 | Throughput (B/s) (Columbia)<br>*Throughput (B/s) (Columbia)* | `curl -w` (`speed_download`) | `...` B/s<br>![C.3-4 speed_download CUBIC](img/c3-04-cubic.png) | `...` B/s<br>![C.3-4 speed_download Reno](img/c3-04-reno.png) | Batas atas teoritis 250.000 B/s (rate 2 Mbps). Analisis: `...` |
| 5 | Waktu transfer total (s) (Columbia)<br>*Total transfer time (s) (Columbia)* | `curl -w` (`time_total`) | `...` s<br>![C.3-5 time_total CUBIC](img/c3-05-cubic.png) | `...` s<br>![C.3-5 time_total Reno](img/c3-05-reno.png) | Batas bawah teoritis sekitar 20 s. Analisis: `...` |
| 6 | Jumlah retransmisi<br>*Number of retransmissions* | `tcp.analysis.retransmission` | `...` paket<br>![C.3-6 retransmisi CUBIC](img/c3-06-cubic.png) | `...` paket<br>![C.3-6 retransmisi Reno](img/c3-06-reno.png) | Retransmisi yang banyak mengindikasikan algoritma bersifat loss-based.<br>*Many retransmissions indicate a loss-based algorithm.*<br>Analisis: `...` |
| 7 | Jumlah duplicate ACK<br>*Number of duplicate ACKs* | `tcp.analysis.duplicate_ack` | `...` paket<br>![C.3-7 duplicate ACK CUBIC](img/c3-07-cubic.png) | `...` paket<br>![C.3-7 duplicate ACK Reno](img/c3-07-reno.png) | Indikasi segmen yang sempat hilang di tengah transfer.<br>*An indication of segments lost during the transfer.*<br>Analisis: `...` |
| 8 | Pola congestion window (awal - akhir)<br>*Congestion window pattern (start - end)* | `tcp.analysis.bytes_in_flight` | Awal `...` B, akhir `...` B, pola: `...`<br>![C.3-8 bytes_in_flight CUBIC](img/c3-08-cubic.png) | Awal `...` B, akhir `...` B, pola: `...`<br>![C.3-8 bytes_in_flight Reno](img/c3-08-reno.png) | Amati pertumbuhan dan stabilitas pada grafik Time - Sequence.<br>*Observe growth and stability in the Time - Sequence graph.*<br>`bytes_in_flight` hanya perkiraan cwnd, bukan pengukuran langsung. Analisis: `...` |
| 9 | Nilai receive window rata-rata (Columbia)<br>*Average receive window value (Columbia)* | `tcp.window_size_value` | `...` B<br>![C.3-9 receive window CUBIC](img/c3-09-cubic.png) | `...` B<br>![C.3-9 receive window Reno](img/c3-09-reno.png) | Menunjukkan kapasitas buffer di sisi penerima.<br>*Shows the buffer capacity on the receiver side.*<br>Analisis: `...` |
| 10 | Stabilitas throughput (grafik)<br>*Throughput stability (graph)* | Throughput Graph | `...`<br>![C.3-10 Throughput Graph CUBIC](img/c3-10-cubic.png) | `...`<br>![C.3-10 Throughput Graph Reno](img/c3-10-reno.png) | Apakah throughput cenderung konstan atau berfluktuasi.<br>*Whether throughput tends to stay constant or fluctuate.*<br>Analisis: `...` |


#### C.3 - Pertanyaan Analisis / Refleksi

> 1. Algoritma mana yang memberikan throughput lebih tinggi pada kondisi jaringan ini? Mengapa demikian?
>    *Which algorithm provides higher throughput under these network conditions? Why?*
>
> 2. Apakah ada hubungan antara jumlah retransmisi dan jenis algoritma congestion control yang digunakan?
>    *Is there a relationship between the number of retransmissions and the congestion control algorithm used?*
>
> 3. Apa perbedaan bentuk kurva congestion window antara Reno dan Cubic pada grafik Time-Sequence (Stevens)?
>    *How do the congestion window curves of Reno and Cubic differ in the Time-Sequence (Stevens) graph?*
>
> 4. Algoritma mana yang paling sensitif terhadap RTT tinggi?
>    *Which algorithm is most sensitive to high RTT?*
>
> 5. Algoritma mana yang lebih efisien pada jaringan dengan latency tinggi seperti pada skenario ini? Jelaskan.
>    *Which algorithm is more efficient on a network with high latency, as in this scenario? Explain.*

**Answer:**

1. Berdasarkan Tabel C.3, `...` memberikan throughput lebih tinggi (`...` B/s dibanding `...` B/s). Penyebabnya: saat terjadi loss, CUBIC hanya menurunkan cwnd sekitar 30% (faktor 0,7) sedangkan Reno menurunkannya 50%, dan CUBIC memulihkan cwnd dengan cepat mendekati nilai sebelum loss, sedangkan Reno hanya menambah sekitar 1 MSS per RTT. Pada RTT sekitar 200 ms, pemulihan linear Reno lambat sehingga pengirim lebih lama berada di bawah kapasitas link. Selisihnya dibatasi oleh rate limit 2 Mbps dan loss acak, sehingga satu kali percobaan dapat bervariasi.

2. Ada hubungan, tetapi bukan jumlah retransmisi yang menentukan algoritma. Keduanya algoritma loss-based, jadi retransmisi terjadi karena loss acak 1% pada jalur, dan jumlahnya ikut bergantung pada berapa banyak segmen yang dikirim selama transfer. Data: CUBIC `...` retransmisi, Reno `...` retransmisi. Perbedaan jumlah tersebut dapat berasal dari perbedaan agresivitas pengiriman dan variasi loss acak antar percobaan: `...`

3. Pada grafik Time-Sequence (Stevens), Reno menunjukkan pola gigi gergaji dengan kenaikan linear yang landai setelah setiap loss. CUBIC menunjukkan kenaikan berbentuk kurva kubik: naik cepat sesudah loss, melandai saat mendekati titik cwnd sebelum loss, lalu naik cepat lagi untuk mencari kapasitas. Pengamatan aktual: `...`. Grafik ini menunjukkan data yang terkirim terhadap waktu, bukan nilai cwnd secara langsung.

4. Reno paling sensitif terhadap RTT tinggi. Reno menambah cwnd 1 MSS per RTT, sehingga makin besar RTT makin lambat cwnd pulih, dan throughputnya berbanding terbalik dengan RTT. CUBIC menumbuhkan cwnd sebagai fungsi waktu sejak loss terakhir, sehingga pertumbuhannya tidak bergantung pada RTT.

5. Secara teori CUBIC lebih efisien pada jaringan dengan latency tinggi karena pertumbuhan cwnd tidak bergantung pada RTT dan pemulihan setelah loss lebih cepat. Pada skenario ini hasil aktualnya: `...`. Pada BDP yang kecil (2 Mbps x sekitar 200 ms = sekitar 50 KB) perbedaan keduanya dapat menyempit, karena CUBIC masuk ke mode TCP-friendly.

---

# Bagian D - Verifikasi Perilaku TCP vs UDP

> [!NOTE]
> Packet loss 5% pada eth1 dan eth2 RhodesIsland. `iperf3 -s` di Laterano; client di Columbia.

## Problem D.1 - D.3

#### Soal D.1 - Setup Gangguan (`tc qdisc show`)

> Loss 5% diterapkan pada kedua interface RhodesIsland.

**Answer:**

Pada RhodesIsland:

```
tc qdisc del dev eth1 root
tc qdisc del dev eth2 root
tc qdisc add dev eth1 root netem loss 5%
tc qdisc add dev eth2 root netem loss 5%
tc qdisc show dev eth1
tc qdisc show dev eth2
```

Kedua interface menampilkan `netem ... loss 5%`.

![tc qdisc show eth1 dan eth2 (loss 5%)](img/d1-tc-qdisc.png)

#### Soal D.3 - Tabel Hasil Pengujian TCP vs UDP

> Jalankan pengujian TCP lalu UDP, catat hasil sender/receiver (nilai + screenshot), lalu bandingkan. Kesimpulan harus didasarkan pada hasil pengukuran yang setara.
> Setiap sel kosong pada tabel di bawah **wajib** berisi **nilai aktual** dan **screenshot** sebagai bukti.
>
> *Run the TCP test and then the UDP test, record the sender/receiver results (value + screenshot), then compare them. The conclusion must be based on comparable measurements.*
> *Every cell in the table below **must** contain the **actual value** and a **screenshot** as evidence.*

Persiapan: iperf3 dipasang dan dijalankan sebagai server pada Laterano.

```
# Laterano
apk add iperf3
iperf3 -s
```

| Parameter | TCP | UDP |
| --- | --- | --- |
| Command di Columbia<br>*Command on Columbia* | `iperf3 -c 10.4.4.15 -t 10` | `iperf3 -c 10.4.4.15 -u -b 5M -t 10` |
| Bitrate/throughput sender | `...` Mbits/sec | `...` Mbits/sec |
| Bitrate/throughput receiver | `...` Mbits/sec | `...` Mbits/sec |
| Retransmission yang dilaporkan<br>*Reported retransmissions* | `...` (kolom Retr) | Tidak ada retransmission bawaan UDP<br>*UDP has no built-in retransmission* |
| Lost / Total Datagrams | Tidak tersedia dalam format TCP<br>*Not available in TCP format* | `...` / `...` (`...`%) |
| Jitter (jika tercantum)<br>*Jitter (if shown)* | — | `...` ms |
| Ringkasan respons terhadap loss<br>*Summary of the response to loss* | Loss dipulihkan lewat retransmisi dan cwnd turun, sehingga throughput menurun, tetapi seluruh data tetap sampai utuh. | Loss tidak ditangani: pengirim tetap mengirim sesuai `-b 5M`, datagram yang hilang tidak dikirim ulang dan dilaporkan sebagai Lost/Total Datagrams. |
| Screenshot output iperf3 | ![D.3 iperf3 TCP](img/d3-iperf-tcp.png) | ![D.3 iperf3 UDP](img/d3-iperf-udp.png) |


#### D.3 - Pertanyaan Analisis / Refleksi

> 1. Mengapa hasil iperf3 mode UDP menampilkan persentase datagram loss, sedangkan hasil mode TCP tidak?
>    *Why do the iperf3 UDP results show the percentage of datagram loss, while the TCP results do not?*
>
> 2. Mengapa throughput UDP tetap mendekati nilai yang diminta (`-b`) meskipun terjadi packet loss, sedangkan throughput TCP menurun?
>    *Why does UDP throughput stay close to the requested value (-b) despite packet loss, while TCP throughput decreases?*
>
> 3. Mengapa throughput TCP bisa turun pada kondisi loss, sedangkan UDP tetap mencoba mengirim sesuai target bitrate?
>    *Why can TCP throughput decrease under loss, while UDP keeps trying to send at the target bitrate?*
>
> 4. Untuk jenis aplikasi seperti apa pilihan UDP lebih tepat dibanding TCP, berdasarkan hasil pengamatan ini?
>    *Based on these observations, for what types of applications is UDP a better choice than TCP?*

**Answer:**

1. Pada mode UDP, iperf3 menyisipkan nomor urut pada tiap datagram, sehingga penerima dapat mengetahui datagram yang tidak pernah tiba dan melaporkannya sebagai Lost/Total Datagrams. UDP sendiri tidak memperbaiki loss, jadi loss tampak apa adanya. Pada mode TCP, loss sudah ditangani oleh protokol lewat retransmisi sehingga aplikasi menerima data yang lengkap; yang tampak hanyalah kolom Retr (jumlah retransmisi) dan perubahan throughput, bukan persentase loss.

2. UDP tidak memiliki umpan balik dan tidak memiliki congestion control. Pengirim terus mengirim pada bitrate yang diminta (5 Mbps) tanpa peduli ada datagram yang hilang. Bitrate sender karena itu mendekati 5 Mbps, sedangkan receiver menerima sekitar 5 Mbps dikurangi bagian yang hilang (sekitar 5%). Pada TCP, loss mengurangi cwnd sehingga laju kirim turun.

3. TCP menganggap loss sebagai tanda congestion, sehingga setiap loss menurunkan cwnd dan memperlambat pengiriman, ditambah waktu untuk retransmisi dan menunggu byte yang hilang terisi. Throughput TCP berbanding terbalik dengan akar tingkat loss (hubungan Mathis), sehingga pada loss 5% throughput turun jauh. UDP tidak punya mekanisme tersebut dan tetap mengirim sesuai target bitrate.

4. UDP lebih tepat untuk aplikasi yang mengutamakan ketepatan waktu dibanding kelengkapan data, dan sanggup mentoleransi sebagian kecil loss: panggilan suara atau video (VoIP, video conference), streaming langsung, game online, serta query singkat seperti DNS. Pada aplikasi seperti itu data yang terlambat tidak berguna lagi, sehingga menunggu retransmisi justru merugikan.

---

# Bagian E - Tugas Mandiri

> [!IMPORTANT]
> Gunakan topologi dari pengerjaan Bagian A - D sebelumnya. Isi tabel dengan **hasil aktual**, bukan angka perkiraan.
>
> *Use the topology from your earlier work in Parts A - D. Fill in the table with **actual results**, not estimated numbers.*

#### Soal E.1 - Ulangi Skenario 1 untuk Tiga Ukuran Berkas

> Ulangi Skenario 1 (tanpa gangguan) untuk `tes1000.bin`, `tes10000.bin`, dan `tes5M.bin`, lalu lengkapi tabel berikut berdasarkan capture Wireshark di Laterano dan Columbia serta output `curl`.
> Setiap sel kosong pada tabel di bawah **wajib** berisi **nilai aktual** dan **screenshot** sebagai bukti.
>
> *Repeat Scenario 1 for the three file sizes and complete the table using the Wireshark captures from Laterano and Columbia and the `curl` output.*
> *Every cell in the table below **must** contain the **actual value** and a **screenshot** as evidence.*

Qdisc gangguan dihapus terlebih dahulu pada RhodesIsland, lalu tiap berkas diunduh dari Columbia dengan capture terpisah:

```
tc qdisc del dev eth1 root
tc qdisc del dev eth2 root
tc qdisc show dev eth1
tc qdisc show dev eth2
```

```
curl -o /dev/null -s -w "size=%{size_download}B speed=%{speed_download}B/s total=%{time_total}s\n" http://10.4.4.15:9999/tes1000.bin
curl -o /dev/null -s -w "size=%{size_download}B speed=%{speed_download}B/s total=%{time_total}s\n" http://10.4.4.15:9999/tes10000.bin
curl -o /dev/null -s -w "size=%{size_download}B speed=%{speed_download}B/s total=%{time_total}s\n" http://10.4.4.15:9999/tes5M.bin
```
 
**Tabel Observasi Kondisi Normal**
 
| No | Parameter/Aspek | Field Wireshark | tes1000.bin | tes10000.bin | tes5M.bin |
| --- | --- | --- | --- | --- | --- |
| 1 | Ukuran berkas (B)<br>*File size (B)* | - | 1000 B | 10000 B | 5000000 B |
| 2 | Jumlah total segmen TCP<br>*Total number of TCP segments* | Statistics Conversations TCP | `...` | `...` | `...` |
| 3 | RTT rata-rata (Laterano)<br>*Average RTT (Laterano)* | `tcp.analysis.ack_rtt` | `...` ms | `...` ms | `...` ms |
| 4 | Throughput (curl) | `speed_download` | `...` B/s | `...` B/s | `...` B/s |
| 5 | Waktu total transfer (curl)<br>*Total transfer time (curl)* | `time_total` | `...` s | `...` s | `...` s |
| 6 | Sequence number awal client<br>*Client's initial sequence number* | `tcp.seq` (SYN) | `...` | `...` | `...` |
| 7 | Sequence number awal server<br>*Server's initial sequence number* | `tcp.seq` (SYN-ACK) | `...` | `...` | `...` |
| 8 | Jumlah ACK yang dikirim client<br>*Number of ACKs sent by the client* | Statistics Endpoints TCP | `...` | `...` | `...` |
| 9 | Status retransmisi (Laterano)<br>*Retransmission status (Laterano)* | `tcp.analysis.retransmission` | `...` | `...` | `...` |
| 10 | Screenshot | Wireshark / curl | ![E.1 tes1000.bin](img/e1-tes1000.png) | ![E.1 tes10000.bin](img/e1-tes10000.png) | ![E.1 tes5M.bin](img/e1-tes5M.png) |

Ukuran berkas diverifikasi pada Laterano dengan `ls -l /root/myscripts/myweb` dan pada output `curl` (`size_download`).

![E.1 ls -l berkas uji](img/e1-ls-berkas.png)

#### E.2 - Pertanyaan Kesimpulan

> 1. Apakah jumlah segmen TCP, RTT, dan throughput berubah secara proporsional terhadap ukuran berkas?
>    *Do the number of TCP segments, RTT, and throughput change in proportion to file size?*
>
> 2. Parameter mana yang relatif tetap tidak bergantung pada ukuran berkas, dan mengapa demikian?
>    *Which parameters stay relatively constant regardless of file size, and why?*
>
> 3. Apa perbedaan antara total waktu download yang diukur curl dengan RTT paket TCP?
>    *What is the difference between the total download time measured by curl and TCP packet RTT?*
>

**Answer:**

1. Jumlah segmen TCP mengikuti ukuran berkas secara hampir proporsional: berkas dipecah menjadi segmen sebesar MSS (sekitar 1448 B pada jalur ini), ditambah segmen tetap untuk handshake, request GET, dan penutupan koneksi. Pada berkas kecil (`tes1000.bin`) bagian tetap ini mendominasi, sehingga rasionya tidak proporsional. RTT tidak proporsional karena ditentukan oleh jalur, bukan ukuran berkas. Throughput juga tidak proporsional: berkas kecil selesai sebelum cwnd sempat tumbuh (slow start) dan waktu handshake ikut terhitung, sehingga throughput `tes1000.bin` jauh lebih rendah dibanding `tes5M.bin`. Data: `...`

2. RTT relatif tetap karena bergantung pada jalur (jumlah hop, delay antrean, dan kecepatan link), yang sama untuk semua berkas. Retransmisi tetap 0 pada kondisi tanpa gangguan. Sequence number awal client dan server tetap acak setiap koneksi karena dibangkitkan kernel dan tidak ada hubungannya dengan ukuran berkas. Pola handshake (SYN, SYN-ACK, ACK) dan penutupan koneksi juga sama.

3. Total waktu `curl` (`time_total`) adalah waktu seluruh transaksi dari awal koneksi sampai data terakhir diterima, mencakup three-way handshake, pengiriman request, waktu proses server, transfer seluruh data (termasuk pertumbuhan cwnd), dan penutupan. RTT paket TCP (`tcp.analysis.ack_rtt`) hanya waktu antara satu segmen dikirim dan ACK-nya diterima. `time_total` bertambah seiring ukuran berkas, sedangkan RTT relatif tetap.
