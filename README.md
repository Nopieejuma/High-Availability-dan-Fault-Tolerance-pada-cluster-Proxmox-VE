# High Availability (HA) dan Fault Tolerance pada Cluster Proxmox VE

Repositori ini berisi dokumentasi, skenario pengujian, dan skrip otomatisasi untuk evaluasi ketahanan sistem **High Availability (HA)** serta **Fault Tolerance (FT)** pada cluster berbasis **Proxmox VE Community Edition** dengan penyimpanan terdistribusi **Ceph Storage**.

---

## 📌 Ringkasan Penelitian

* **Judul Proyek Akhir:** High Availability (HA) dan Fault Tolerance pada Cluster Proxmox VE
* **Penulis:** Novia Zulma (NIM. 2355301165)
* **Pembimbing:** Muhammad Arif Fadhly Ridha, S.Kom., M.T.
* **Institusi:** Program Studi Teknik Informatika, Politeknik Caltex Riau (2025/2026)

---

## 🛠️ Arsitektur & Spesifikasi Perangkat

Sistem dibangun menggunakan **3 Server Fisik (Bare-Metal)** yang membentuk satu cluster Proxmox VE terintegrasi dengan Ceph Storage sebagai *shared storage* terdistribusi.

### Spesifikasi Node Cluster
| Perangkat | Role / Fungsi | CPU | RAM | Storage | IP Address | OS & Version |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **S01** | Primary Node, Ceph MON & OSD.0 | Intel Core i7-14700F | 32 GB | 1 TB HDD (`/dev/sda4`) | `160.187.144.188` | Proxmox VE 9.1 / Ceph Squid 19.2 |
| **S02** | Standby Node, Ceph MON & OSD.1 | Intel Core i7-14700F | 32 GB | 1 TB HDD (`/dev/sda4`) | `160.187.144.189` | Proxmox VE 9.1 / Ceph Squid 19.2 |
| **S03** | Standby Node / Quorum, Ceph MON & OSD.2 | Intel Core i7-14700F | 32 GB | 1 TB HDD (`/dev/sda4`) | `160.187.144.190` | Proxmox VE 9.1 / Ceph Squid 19.2 |
| **VM Active** | Guest OS (Mail & FTP Server) | 2 Cores | - | 20 GB (Ceph Pool) | `160.187.144.187` | Ubuntu Live Server 24.04 LTS |
| **PC Client** | Workstation & Script Monitoring | Intel Core i7-14700F | 32 GB | 1 TB HDD | `172.19.10.220` | Windows 11 Home |

---

## ⚙️ Skenario Pengujian

1. **Skenario 1: Power Failure (Node Shutdown)**
   * **Deskripsi:** Simulasi pemutusan daya mendadak dengan mencabut kabel power pada node aktif yang sedang menjalankan VM (`S01`).
   * **Tujuan:** Mengukur rata-rata durasi *downtime* layanan dan kecepatan *failover* otomatis ke node sehat (`S02`/`S03`).

2. **Skenario 2: Disk Failure (Ceph OSD Failure)**
   * **Deskripsi:** Pelepasan/penghentian paksa OSD pada penyimpanan terdistribusi Ceph saat layanan FTP dan Mail Server beroperasi.
   * **Tujuan:** Menguji fitur *Fault Tolerance* Ceph Storage dan mengukur durasi pemulihan data (*Recovery Time*) hingga cluster kembali ke status `HEALTH_OK`.

---

## 📊 Hasil Pengujian & Evaluasi Performa

### 1. Pengujian High Availability (Power Failure)
Pengujian dilakukan sebanyak 15 kali iterasi otomatisasi menggunakan Script Bash dan PowerShell ICMP Timestamping:

* **Rata-Rata Failover Time:** **110 Detik**  
  *(Lebih cepat dari standar baseline Proxmox HA Manager sebesar 120 detik, tergolong **Sangat Bagus**)*
* **Rata-Rata Downtime Layanan:** **186 Detik**  
  *(Termasuk durasi booting ulang internal Guest OS Ubuntu Server, tergolong **Bagus**)*

### 2. Pengujian Fault Tolerance (Disk Failure & Ceph Recovery)
* **Rata-Rata Ceph Recovery Time:** **160 Detik** *(Status kembali ke `HEALTH_OK`)*
* **Dampak Performa Layanan Saat Recovery:**
  * **FTP Server:** File `< 400 MB` berhasil diunggah dengan peningkatan *latency*. File `> 400 MB` mengalami *timeout* akibat perebutan bandwidth I/O (*I/O contention*).
  * **Mail Server:** Mengalami *downtime* total/pemblokiran transaksi pesan selama proses *rebalancing* data Ceph karena tingginya sensitivitas terhadap *I/O latency*.

### 3. Matriks Komparasi Kondisi System
| Kondisi Sistem | Rata-rata Downtime | Rata-rata Failover | Rata-rata Recovery Ceph | Status Layanan Pasca Failure |
| :--- | :--- | :--- | :--- | :--- |
| **Ceph + HA Aktif** | **186 detik** | **110 detik** | **160 detik** | Pulih Otomatis pada Node Sehat |
| **Ceph Tanpa HA** | N/A | N/A | **143 detik** | VM Mati Total (Manual Recovery) |
| **Tanpa Ceph & Tanpa HA** | N/A | N/A | N/A | Lumpuh Total / Isolasi Data |

---

## 💻 Skrip Otomatisasi Monitoring

Pengukuran parameter dilakukan secara presisi dengan skrip otomatis:

1. **Script Downtime (ICMP Ping Monitoring pada Client):**  
   Mencatat selisih timestamp antara respon *Request Time Out* (RTO) pertama hingga respon *Reply* pertama diterima kembali.
2. **Script Failover (`pve-ha-crm` Monitor):**  
   Menghitung selisih timestamp saat kegagalan node terdeteksi hingga VM berstatus `started` di node baru.
3. **Script Recovery Ceph (`ceph -s` Monitor):**  
   Mencatat waktu dari penghentian OSD/degraded status hingga cluster kembali ke status `HEALTH_OK`.

---

## 📝 Kesimpulan & Saran

### Kesimpulan
1. Cluster 3 node berbasis Proxmox VE Community Edition dan Ceph Storage berhasil mengeksekusi *failover* otomatis tanpa kehilangan data (*zero data loss*).
2. Batas maksimum *Fault Tolerance* cluster 3 node adalah **1 node failure**. Jika 2 node mati bersamaan, cluster kehilangan *Quorum* (< 50% voting), memicu penguncian proteksi (*freeze*) demi mencegah korupsi data.
3. Disk failure pada Ceph tidak mematikan cluster, namun memicu degradasi performa (*I/O contention*) pada layanan yang sensitif *latency* seperti Mail Server.

### Saran Pengembangan
* Pisahkan disk untuk OS Proxmox VE dan Ceph OSD pada drive fisik terpisah untuk menghindari *I/O bottleneck*.
* Gunakan *Dedicated High-Speed Network* (misal: 10Gbps atau Network Bonding) khusus untuk alur *replication & rebalancing* Ceph.
* Tambahkan jumlah node menjadi minimal **5 node** untuk meningkatkan ketahanan toleransi kegagalan hingga 2 node sekaligus tanpa kehilangan quorum.
