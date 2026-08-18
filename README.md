# High-Availability-dan-Fault-Tolerance-pada-cluster-Proxmox-VE

# Implementation & Benchmark of High Availability (HA) and Fault Tolerance (FT) on Proxmox VE Cluster with Ceph Storage

Repositori ini berisi dokumentasi arsitektur, skrip otomatisasi pengujian, file konfigurasi, dan data hasil uji coba *failover* serta *recovery* pada kluster Proxmox VE berbasis Ceph Storage.

---

## 📌 Deskripsi Proyek
Proyek ini mengimplementasikan infrastruktur *High Availability* (HA) dan *Fault Tolerance* (FT) menggunakan 3 node server fisik yang terdistribusi secara terpusat dengan *shared storage* Ceph. Layanan yang dijalankan di atas kluster ini adalah **Mail Server** dan **FTP Server** untuk menguji ketahanan sistem saat terjadi kegagalan (*node crash* maupun *disk failure*).

---

## 📐 Topologi & Arsitektur Sistem

> 💡 *Upload gambar diagram topologi Anda ke folder docs/ lalu ubah link gambar di bawah ini.*

![Architecture Diagram](docs/architecture.png)

* **Jumlah Node:** 3 Server Fisik (Node 1, Node 2, Node 3)
* **Storage Cluster:** Ceph HCI (Hyper-Converged Infrastructure)
* **Dedicated Network:** 
  * Public Network (Manajemen & Akses Layanan)
  * Cluster Network (Sinkronisasi Ceph & Corosync)
* **Layanan Utama:** Mail Server & FTP Server

---

## 🖥️ Spesifikasi Perangkat (Hardware & Software)

| Komponen | Spesifikasi / Versi |
| :--- | :--- |
| **Hypervisor** | Proxmox VE v8.x |
| **Storage Engine** | Ceph (Reef / Quincy) |
| **Processor (CPU)** | Intel / AMD [Sebutkan model/core] |
| **Memory (RAM)** | [Sebutkan total RAM per node, misal: 32GB] |
| **Disk (OSD)** | [Sebutkan jenis SSD/HDD per node] |
| **Network Interface** | [Sebutkan speed NIC, misal: Dual 1Gbps / 10Gbps] |

---

## 📊 Hasil Pengujian Kegagalan (Failover & Recovery)

Data berikut diperoleh menggunakan **skrip pengukur otomatis** yang tersimpan pada direktori `/scripts`. Pengujian dilakukan secara nyata (*real failure test*).

### 1. Skenario Uji: Cabut Power Mendadak (Node Crash / HA Test)
*Pengujian dilakukan dengan mencabut kabel daya salah satu node utama secara mendadak.*

| Layanan | Status Sebelum Failure | Waktu Failover (Downtime) | Status Pasca Failover |
| :--- | :--- | :--- | :--- |
| **Mail Server** | Running (Node 1) | **X.X Detik** | Running Auto (Node 2) |
| **FTP Server** | Running (Node 1) | **X.X Detik** | Running Auto (Node 3) |

### 2. Skenario Uji: Cabut Storage / Disk OSD (Fault Tolerance & Ceph Recovery Test)
*Pengujian dilakukan dengan mencabut salah satu HDD/SSD Ceph OSD pada node aktif.*

| Parameter Pengujian | Hasil Pengukuran | Keterangan |
| :--- | :--- | :--- |
| **Downtime Layanan (Mail & FTP)** | **0 Detik** | Layanan tetap berjalan lancar (*Zero Downtime*) |
| **Waktu Recovery Ceph (Rebalance)** | **X.X Menit / Detik** | Waktu yang dibutuhkan Ceph untuk *degraded OSD rebalancing* |

---

## 📂 Struktur Repository

```text
.
├── README.md               # Dokumentasi utama proyek
├── docs/                   # Gambar diagram dan arsitektur
│   └── architecture.png
├── scripts/                # Skrip pengukur otomatis
│   ├── measure-failover.sh # Skrip pengukur downtime / failover
│   └── ceph-recovery.sh    # Skrip pengukur waktu recovery Ceph
└── configs/                # File konfigurasi layanan
    ├── mail/               # File konfigurasi Mail Server
    └── ftp/                # File konfigurasi FTP Server
