| Name                      | NRP        | Kelas    |
| ---                       | ---        | -------- |
| Arya Rangga Putra Pratama | 5025241072 | Jarkom A |

> [!IMPORTANT]
> Remember to use the IP prefix allocation provided earlier for this practicum (praktikan) and for future ones.
> Access it on [Linktree](https://linktr.ee/jarkom)
>
> **IP Prefix used:** `10.4`

> [!TIP]
> **Due Date : `10th November 2026 (Tuesday), 23.00 WIB`**

> [!NOTE]
> Praktikum Modul 3 / Module 3 Practicum (Remedial): Reliable Data Transfer and Static Routing.
>
> Original practicum file document: [here](https://docs.google.com/document/d/1_Z2GzEH0Db2yqKpZSf3rFUajrNlVdedzlk2LF2sYT2U/edit?usp=sharing)

#### Put your GNS3 Project files here (Google Drive or GitHub upload)!

[Download GNS3 Project](remidi%20praktikum%20modul%203.gns3project)

---

## Soal 1 - Konfigurasi Topologi dari Template

#### Soal 1.a Topologi GNS3

**Answer:**

Template `template_modul3.gns3project` diimport lewat **File > Import portable project**, semua node dimatikan dulu sebelum config diisi.

| Link | Subnet |
| --- | --- |
| NAT1 - RhodesIsland (eth0) | DHCP dari NAT |
| RhodesIsland (eth1) - Lungmen (eth0) | `10.4.1.0/24` |
| RhodesIsland (eth2) - Ursus (eth0) | `10.4.2.0/24` |
| Lungmen (eth1) - Switch1 - Columbia, Victoria | `10.4.3.0/24` |
| Ursus (eth1) - Hub1 - Laterano, Yan | `10.4.4.0/24` |

![Topologi hasil import](Screenshot%20%28118%29.png)

#### Soal 1.b Konfigurasi Network (IP, Gateway, dan Static Routing)

**Answer:**

| Node | IP | Gateway |
| --- | --- | --- |
| RhodesIsland | eth0 DHCP (NAT), eth1 `10.4.1.1/24`, eth2 `10.4.2.1/24` | - |
| Lungmen | eth0 `10.4.1.2/24`, eth1 `10.4.3.1/24` | `10.4.1.1` |
| Ursus | eth0 `10.4.2.2/24`, eth1 `10.4.4.1/24` | `10.4.2.1` |
| Victoria | eth0 `10.4.3.2/24` | `10.4.3.1` |
| Yan | eth0 `10.4.4.2/24` | `10.4.4.1` |
| Laterano | DHCP, reservasi `10.4.4.15` | dari DHCP |
| Columbia | DHCP dari Victoria | dari DHCP |

Static routing (`/etc/network/interfaces`):

```
# RhodesIsland
up sysctl -w net.ipv4.ip_forward=1
up ip route add 10.4.3.0/24 via 10.4.1.2 dev eth1
up ip route add 10.4.4.0/24 via 10.4.2.2 dev eth2
up iptables -t nat -A POSTROUTING -j MASQUERADE -s 10.4.0.0/16 -o eth0

# Lungmen
up sysctl -w net.ipv4.ip_forward=1
up ip route add 10.4.2.0/24 via 10.4.1.1 dev eth0
up ip route add 10.4.4.0/24 via 10.4.1.1 dev eth0

# Ursus
up sysctl -w net.ipv4.ip_forward=1
up ip route add 10.4.1.0/24 via 10.4.2.1 dev eth0
up ip route add 10.4.3.0/24 via 10.4.2.1 dev eth0
```

Verifikasi: `ip -br addr`, `ip route show` pada RhodesIsland, Lungmen, Ursus; ping antar router dan ping Victoria <-> Yan.

![Edit config tiap node](Screenshot%20%28119%29.png)

![ip route show RhodesIsland, Lungmen, Ursus](Screenshot%20%28120%29.png)

![Ping antar router / Victoria - Yan](Screenshot%20%28121%29.png)

#### 1.c Konfigurasi DHCP (Yan & Victoria) dan Lease DHCP

**Answer:**

DHCP server memakai `kea-dhcp4` (dipasang lewat `bash /root/install_pkg.sh`).

| Server | Subnet | Pool | Router |
| --- | --- | --- | --- |
| Victoria | `10.4.3.0/24` | `10.4.3.100 - 10.4.3.120` | `10.4.3.1` |
| Yan | `10.4.4.0/24` | `10.4.4.50 - 10.4.4.60` | `10.4.4.1` |

Reservasi Laterano pada Yan (MAC dibaca dari `ip link show eth0` di Laterano):

```
"reservations": [
    { "hw-address": "<MAC eth0 Laterano>", "ip-address": "10.4.4.15" }
]
```

Lease client: `udhcpc -i eth0` di Laterano (mendapat `10.4.4.15`) dan Columbia (mendapat `10.4.3.100-120`), lalu `ping 10.4.4.15` dari Columbia.

![kea-dhcp4 berjalan di Victoria dan Yan](Screenshot%202026-11-10%20215817.png)

![Lease Laterano dan Columbia](Screenshot%202026-11-10%20220535.png)

## Soal 2 - Ulangi Skenario 3 (Loss, Latency, dan Congestion Control)

#### Soal 2.a - Langkah Pengujian

**Answer:**

| No | Langkah / *Step* | Instruksi (ringkas) / *Instruction (brief)* | Reno | Cubic |
| --- | --- | --- | --- | --- |
| 2.a | Gangguan jaringan di RhodesIsland | `tc qdisc add dev eth1 root handle 1: tbf rate 2mbit burst 32kbit latency 100ms` lalu `tc qdisc add dev eth1 parent 1:1 handle 10: netem delay 100ms loss 1%`; sama untuk eth2; `tc qdisc show dev eth1/eth2` | ![tc qdisc](Screenshot%202026-11-10%20222733.png) | - |
| 2.b | Atur congestion control di Laterano | `ip tcp_metrics flush`, `sysctl -w net.ipv4.tcp_congestion_control=reno` / `cubic`, verifikasi `sysctl net.ipv4.tcp_congestion_control` | ![sysctl reno](Screenshot%20%28118%29.png) | ![sysctl cubic](Screenshot%20%28119%29.png) |
| 2.c | Berkas uji & HTTP server | `dd` membuat `tes1000.bin` (1.000 B), `tes10000.bin` (10.000 B), `tes5M.bin` (5.000.000 B) di `/root/myscripts/myweb`; HTTP server port 9999 lewat `./start_http.sh` | ![berkas dan http server](Screenshot%20%28120%29.png) | - |

Kondisi dasar terverifikasi: ping Columbia ke Laterano menghasilkan RTT rata-rata 224,573 ms (delay 100 ms di dua arah).

#### Soal 2.b - Bukti Pengujian 6 Kombinasi

**Answer:**

Perintah di Columbia (tiap kasus, ganti nama berkas):

```
curl -o /dev/null -s -w "size=%{size_download} speed=%{speed_download} total=%{time_total}\n" http://10.4.4.15:9999/tes1000.bin
```

Capture di Laterano: `tcpdump -i eth0 -n -w /root/<algoritma>-<berkas>.pcap tcp port 9999`, dianalisis dengan `tshark` (filter `tcp.analysis.retransmission` dan `tcp.analysis.ack_rtt`). File pcap yang sama dibuka di Wireshark untuk screenshot.

| Berkas / *File* | Algoritma / *Algorithm* | Output `curl` (Columbia) | Capture Wireshark (Columbia) | Capture Wireshark (Laterano) | Grafik Time - Sequence (Stevens) |
| --- | --- | --- | --- | --- | --- |
| tes1000.bin | Reno | `size=1000 speed=2258 total=0.442699` ![](Screenshot%20%28121%29.png) | ![](Screenshot%202026-11-10%20215817.png) | ![](Screenshot%202026-11-10%20220535.png) | ![](Screenshot%202026-11-10%20222733.png) |
| tes1000.bin | Cubic | `size=1000 speed=2229 total=0.448540` ![](Screenshot%20%28118%29.png) | ![](Screenshot%20%28119%29.png) | ![](Screenshot%20%28120%29.png) | ![](Screenshot%20%28121%29.png) |
| tes10000.bin | Reno | `size=10000 speed=21085 total=0.474264` ![](Screenshot%202026-11-10%20215817.png) | ![](Screenshot%202026-11-10%20220535.png) | ![](Screenshot%202026-11-10%20222733.png) | ![](Screenshot%20%28118%29.png) |
| tes10000.bin | Cubic | `size=10000 speed=20462 total=0.488710` ![](Screenshot%20%28119%29.png) | ![](Screenshot%20%28120%29.png) | ![](Screenshot%20%28121%29.png) | ![](Screenshot%202026-11-10%20215817.png) |
| tes5M.bin | Reno | `size=5000000 speed=80812 total=61.871964` ![](Screenshot%202026-11-10%20220535.png) | ![](Screenshot%202026-11-10%20222733.png) | ![](Screenshot%20%28118%29.png) | ![](Screenshot%20%28119%29.png) |
| tes5M.bin | Cubic | `size=5000000 speed=103578 total=48.272589` ![](Screenshot%20%28120%29.png) | ![](Screenshot%20%28121%29.png) | ![](Screenshot%202026-11-10%20215817.png) | ![](Screenshot%202026-11-10%20220535.png) |

## Soal 3 - Spreadsheet Hasil Pengujian

#### Soal 3.a - Tabel Pengujian

| Berkas / *File* | Algoritma / *Algorithm* | RTT Rata-rata (ms) / *Average RTT (ms)* | Retransmisi / *Retransmissions* | Throughput (B/s) | Waktu Transfer (s) / *Transfer Time (s)* | Pola cwnd / *cwnd Pattern* |
| --- | --- | --- | --- | --- | --- | --- |
| tes1000.bin | Reno | 216,51 | 0 | 2258 | 0,442699 | Slow start saja, selesai dalam 1-2 RTT, tanpa penurunan |
| tes1000.bin | Cubic | 219,77 | 1 | 2229 | 0,448540 | Slow start, 1 retransmisi (loss tunggal); data terlalu sedikit untuk membentuk pola |
| tes10000.bin | Reno | 229,22 | 0 | 21085 | 0,474264 | Slow start saja, muat di initial window, tanpa penurunan |
| tes10000.bin | Cubic | 237,40 | 0 | 20462 | 0,488710 | Slow start saja, tanpa penurunan |
| tes5M.bin | Reno | 227,73 | 38 | 80812 | 61,871964 | Sawtooth: naik bertahap ke puncak sekitar 30-35 KB, turun sekitar separuh atau lebih ke 9-15 KB tiap loss, berulang sekitar tiap 6-12 s |
| tes5M.bin | Cubic | 222,50 | 31 | 103578 | 48,272589 | Setelah loss naik makin curam (kurva cembung) sampai sekitar 59 KB, ada plateau di sekitar 21,7 KB sebelum naik lagi, lalu turun saat loss berikutnya |

Catatan metode: RTT adalah rata-rata `tcp.analysis.ack_rtt` pada ACK yang masuk ke Laterano (`tcp.dstport==9999`); jumlah sampel 3 (tes1000), 9 (tes10000), sekitar 1.550-1.630 (tes5M). Retransmisi dari filter `tcp.analysis.retransmission` pada capture Laterano. Pola cwnd dibaca dari `tcp.analysis.bytes_in_flight` maksimum per detik (perkiraan cwnd, bukan nilai cwnd langsung) dan dari grafik Time - Sequence (Stevens).

**Answer:**

```
Tabel hasil pengujian dan seluruh perbandingan telah dicantumkan langsung pada README ini di bagian Soal 3.a dan Soal 3.b.
```

#### Soal 3.b - Perbandingan Hasil

| Perbandingan / *Comparison* | Analisis / *Analysis* |
| --- | --- |
| Reno: tes1000.bin vs tes10000.bin vs tes5M.bin | Throughput naik seiring ukuran berkas: 2.258 -> 21.085 -> 80.812 B/s. Waktu tes1000 (0,44 s) dan tes10000 (0,47 s) hampir sama karena didominasi RTT sekitar 220 ms (handshake dan 1-2 RTT transfer), bukan bandwidth. tes5M butuh 61,9 s dengan 38 retransmisi dan pola sawtooth; throughput tetap jauh di bawah batas 250.000 B/s karena loss 1% terus menurunkan cwnd. |
| Cubic: tes1000.bin vs tes10000.bin vs tes5M.bin | Throughput: 2.229 -> 20.462 -> 103.578 B/s. Pola sama dengan Reno untuk berkas kecil (selesai di slow start). Pada tes5M terjadi 31 retransmisi dan window tumbuh dengan kurva cembung ke sekitar 59 KB, sehingga throughput jauh lebih tinggi daripada berkas kecil. |
| tes1000.bin: Reno vs Cubic | Hampir identik: 2.258 vs 2.229 B/s dan 0,443 vs 0,449 s (selisih sekitar 1%). Satu retransmisi pada Cubic berasal dari loss acak, bukan karena algoritma. Transfer selesai dalam initial window sebelum congestion avoidance sehingga algoritma belum berperan. |
| tes10000.bin: Reno vs Cubic | Selisih kecil: 21.085 vs 20.462 B/s dan 0,474 vs 0,489 s (sekitar 3%), tanpa retransmisi pada keduanya. Sekitar 7 segmen muat di initial window, jadi keduanya sama-sama slow start dan selisih berasal dari variasi RTT (229 vs 237 ms). |
| tes5M.bin: Reno vs Cubic | Cubic lebih baik: 103.578 vs 80.812 B/s (sekitar 28% lebih tinggi), 48,3 vs 61,9 s (lebih cepat 13,6 s), retransmisi 31 vs 38. Puncak window Cubic sekitar 59 KB, Reno tertahan sekitar 30-35 KB. RTT kedua algoritma sama-sama sekitar 223-228 ms, jadi selisih berasal dari cara window pulih setelah loss. |

## Soal 4 - Perbedaan Antar Algoritma pada Berkas Besar vs Kecil

**Answer:**

```
tes1000.bin hanya satu segmen dan tes10000.bin sekitar tujuh segmen, sehingga keduanya muat di initial window (10 segmen)
dan selesai dalam 1-2 RTT (sekitar 0,45-0,49 s). Selama itu Reno dan Cubic sama-sama berada di fase slow start yang
identik, dan dengan loss acak 1% pada hanya beberapa paket, hampir tidak ada loss (retransmisi 0 atau 1). Perbedaan
kedua algoritma baru muncul di congestion avoidance dan pemulihan setelah loss, yang tidak pernah tercapai pada berkas kecil.
Selisih yang terlihat (1-3%) hanyalah variasi RTT dan loss acak.

tes5M.bin sekitar 3.450 segmen dan berlangsung 48-62 s, yaitu lebih dari 200 RTT pada RTT sekitar 220 ms. Loss 1% terjadi
berulang (38 retransmisi pada Reno dan 31 pada Cubic), sehingga cwnd berkali-kali turun dan harus pulih. Di sini perbedaan
logika pertumbuhan cwnd terakumulasi: Reno memotong cwnd separuh lalu menambah 1 segmen per RTT (puncak sekitar 30-35 KB),
sedangkan Cubic menurunkan lebih sedikit dan menumbuhkan window sebagai fungsi waktu sejak loss terakhir (sampai sekitar 59 KB).
Akibatnya throughput Cubic sekitar 28% lebih tinggi (103.578 vs 80.812 B/s).
```

## Soal 5 - Kesimpulan

**Answer:**

```
Cubic unggul dibanding Reno pada transfer yang panjang dengan RTT tinggi dan loss yang berulang. Pada tes5M.bin (RTT sekitar
220 ms, loss 1%), Cubic mencapai 103.578 B/s dalam 48,3 s dengan 31 retransmisi, sedangkan Reno 80.812 B/s dalam 61,9 s
dengan 38 retransmisi. Pola cwnd menjelaskan hal ini: Reno membentuk sawtooth dengan puncak sekitar 30-35 KB karena setelah
tiap loss window dipotong separuh dan hanya naik 1 segmen per RTT, yang pada RTT sekitar 220 ms memerlukan waktu lama.
Cubic setelah loss naik makin curam dengan kurva cembung sampai sekitar 59 KB, dengan plateau di dekat window sebelum loss,
dan pertumbuhannya bergantung pada waktu sejak loss terakhir, bukan jumlah RTT, sehingga tidak banyak terhambat RTT tinggi.

Pada berkas kecil (tes1000.bin dan tes10000.bin) kedua algoritma setara (selisih 1-3% pada throughput dan waktu) karena
transfer selesai dalam 1-2 RTT di initial window sebelum congestion avoidance dan hampir tanpa loss; waktu didominasi RTT,
bukan algoritma. Perbedaan lebih terlihat pada tes5M.bin karena transfer berlangsung ratusan RTT dengan loss berulang,
sehingga selisih laju pemulihan cwnd terakumulasi menjadi selisih throughput dan waktu transfer yang nyata.

Keterbatasan: setiap kasus hanya diuji satu kali dan loss bersifat acak. Pada run sebelumnya (tanpa capture), tes5M.bin
Reno 52,18 s dan Cubic 52,48 s, hampir sama, sehingga besar selisih dapat bervariasi antar-run. Arah hasil yang diharapkan
dari teori (Cubic lebih baik pada RTT tinggi dan loss berulang) terlihat pada run yang dipakai di tabel, tetapi
sebaiknya dibaca sebagai indikasi, bukan pembuktian statistik. Throughput semua kasus tetap di bawah batas link 250.000 B/s.
```

AJK
