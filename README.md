# 🖥️ Implementasi High Availability Server dengan Load Balancing

**PjBL Jaringan Komputer dan Komunikasi Data — UMRAH 2025/2026**

---

Link Google Drive : https://drive.google.com/drive/folders/12GyOH1DN5AD2LEp-Jh0fNOWwyDldZELP?usp=sharing
Link Demo YouTube : https://youtu.be/1R6A4IEYPPM

---

## 📋 Deskripsi Proyek

Proyek ini merupakan implementasi **High Availability (HA) Server dengan Load Balancing** menggunakan HAProxy sebagai load balancer dan dua web server backend berbasis Apache2. Tujuan utama proyek ini adalah memastikan layanan web tetap berjalan meskipun salah satu server mengalami kegagalan (*failover*).

Infrastruktur dibangun menggunakan **VirtualBox** untuk VM dan **GNS3** untuk simulasi jaringan, dengan topologi yang terdiri dari Router Cisco c3725, Switch, PC Client (VPCS), dan tiga VM Ubuntu Server (HAProxy, WebServer1, WebServer2).

Load balancing menggunakan metode **Round Robin**, sehingga trafik didistribusikan secara bergantian ke WebServer1 dan WebServer2. Ketika salah satu server mati, HAProxy secara otomatis mendeteksi melalui mekanisme **health check** dan mengalihkan seluruh trafik ke server yang masih aktif.

---

## 👥 Anggota Kelompok 7

| No | Nama | NIM |
|----|------|-----|
| 1 | Muhammad Adib Haryadi | 2401020076 |
| 2 | Dhiya Zarifah | 2401020075 |
| 3 | Nisrina Retnosari | 2401020060 |
| 4 | Giffari Zakka Wali Andry | 2401020088 |
| 5 | Muhammad Fathir | 2401020082 |

---

## 🏗️ Topologi Jaringan

```
[PC1/VPCS] ── [Switch1] ── [Router R1] ── [Switch2] ── [HAProxy VM]
  192.168.10.2              f0/0: 192.168.10.1            192.168.56.10
                            f1/0: 192.168.56.1        ├── [WebServer1 VM]
                                                      │    192.168.56.11
                                                      └── [WebServer2 VM]
                                                           192.168.56.12
```

**Alur Trafik Normal:**
> PC1 → Router R1 → HAProxy → WebServer1 / WebServer2 (bergantian)

**Alur Trafik saat Failover:**
> PC1 → Router R1 → HAProxy → WebServer2 (otomatis saat WS1 mati)

---

## 🖥️ Teknologi yang Digunakan

- **GNS3** v2.2.58 — Simulasi topologi jaringan
- **VirtualBox** — Platform virtualisasi VM
- **Ubuntu Server** 22.04 LTS — OS untuk semua VM
- **HAProxy** 2.4.30 — Load balancer & health check
- **Apache2** — Web server di WS1 dan WS2
- **Cisco IOS** c3725 — Router simulasi di GNS3
- **VPCS** — Client network di GNS3

---

## 📊 IP Address Plan

### Tabel Subnet

| No | Nama Subnet | Network Address | Subnet Mask | CIDR |
|----|-------------|-----------------|-------------|------|
| 1 | Client Network | 192.168.10.0 | 255.255.255.0 | /24 |
| 2 | Server Network | 192.168.56.0 | 255.255.255.0 | /24 |

### Tabel IP Per Perangkat

| No | Perangkat | Interface | IP Address | Gateway |
|----|-----------|-----------|------------|---------|
| 1 | Router R1 | f0/0 | 192.168.10.1 | — |
| 2 | Router R1 | f1/0 | 192.168.56.1 | — |
| 3 | HAProxy LB | enp0s3 | 192.168.56.10 | 192.168.56.1 |
| 4 | Web Server 1 | enp0s3 | 192.168.56.11 | 192.168.56.1 |
| 5 | Web Server 2 | enp0s3 | 192.168.56.12 | 192.168.56.1 |
| 6 | PC1 (VPCS) | e0 | 192.168.10.2 | 192.168.10.1 |

---

## ⚙️ Konfigurasi HAProxy

File konfigurasi HAProxy (`/etc/haproxy/haproxy.cfg`) — bagian yang ditambahkan:

```cfg
frontend web_front
    bind *:80
    default_backend web_servers

backend web_servers
    balance roundrobin
    option httpchk GET /
    server webserver1 192.168.56.11:80 check
    server webserver2 192.168.56.12:80 check
```

---

## 📁 Struktur Repository

```
📁 Kelompok7-HighAvailability/
│
├── 📁 docs/
│   ├── 📁 progress-1/
│   │   └── Laporan_P1.pdf
│   ├── 📁 progress-2/
│   │   └── Laporan_P2.pdf
│   └── 📁 progress-3/
│       └── Laporan_P3.pdf
│
├── 📁 config/
│   ├── haproxy.cfg
│   └── netplan-config.yaml
│
├── 📁 gns3/
│   └── HighAvailability.gns3
│
├── 📁 screenshots/
│   ├── topologi-gns3.png
│   ├── ping-test.png
│   ├── curl-roundrobin.png
│   ├── failover.png
│   ├── haproxy-status.png
│   ├── routing-table.png
│   └── traceroute.png
│
└── README.md
```

---

## 🧪 Cara Instalasi dan Konfigurasi

### 1. Install Apache2 di WebServer1 dan WebServer2
```bash
sudo apt update && sudo apt install apache2 -y
echo "<h1>WEB SERVER 1 - Kelompok 7 UMRAH</h1>" | sudo tee /var/www/html/index.html
```

### 2. Install HAProxy
```bash
sudo apt update && sudo apt install haproxy -y
```

### 3. Konfigurasi IP Statis (Netplan)
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.56.10/24
      gateway4: 192.168.56.1
      nameservers:
        addresses:
          - 8.8.8.8
```

### 4. Konfigurasi Router R1 (Cisco IOS)
```
enable
configure terminal
interface f0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
interface f1/0
ip address 192.168.56.1 255.255.255.0
no shutdown
exit
end
write memory
```

---

## 🔬 Pengujian

### Test Load Balancing (Round Robin)
```bash
# Jalankan beberapa kali dari HAProxy terminal
curl http://192.168.56.10
# Output bergantian: WEB SERVER 1 → WEB SERVER 2 → WEB SERVER 1 → ...
```

### Test Failover
```bash
# 1. Matikan WebServer1
sudo poweroff  # (di terminal WS1)

# 2. Curl dari HAProxy
curl http://192.168.56.10
# Output: selalu WEB SERVER 2 (failover berhasil)
```

### Test Ping dari PC1 (VPCS di GNS3)
```
ping 192.168.10.1   # Router
ping 192.168.56.10  # HAProxy
ping 192.168.56.11  # WebServer1
ping 192.168.56.12  # WebServer2
```

### Traceroute dari PC1
```
trace 192.168.56.10
# Hop 1: 192.168.10.1 (Router R1)
# Hop 2: 192.168.56.10 (HAProxy)
```

---

## 📈 Hasil Pengujian

| Pengujian | Hasil |
|-----------|-------|
| Ping PC1 → Router | ✅ Berhasil |
| Ping PC1 → HAProxy | ✅ Berhasil |
| Ping PC1 → WebServer1 | ✅ Berhasil |
| Ping PC1 → WebServer2 | ✅ Berhasil |
| Load Balancing Round Robin | ✅ Bergantian WS1-WS2 |
| Failover WS1 → WS2 | ✅ Otomatis |
| Availability saat failover | ✅ 100% (0 packet loss) |

---

## 📅 Jadwal Progres

| Progres | Deadline | Status |
|---------|----------|--------|
| P1 — Desain Topologi & IP Plan | 13 Juni 2026 | ✅ Selesai |
| P2 — Routing Dasar | 20 Juni 2026 | ✅ Selesai |
| P3 — Load Balancing & Failover | 27 Juni 2026 | ✅ Selesai |
| Final — Analisis Availability | 01-03 Juli 2026 | 🔄 On Progress |

---

## 📚 Referensi

1. HAProxy Documentation — https://www.haproxy.org/documentation/
2. Ubuntu Server Guide — https://ubuntu.com/server/docs
3. GNS3 Documentation — https://docs.gns3.com
4. Tanenbaum, A.S. & Wetherall, D. (2011). *Computer Networks* (5th ed.). Pearson.
5. RFC 2391 — Load Sharing using IP Network Address Translation

---

> **Mata Kuliah:** Jaringan Komputer   
> **Program Studi:** Teknik Informatika  
> **Universitas:** Universitas Maritim Raja Ali Haji (UMRAH)  
> **Tahun Akademik:** 2025/2026
