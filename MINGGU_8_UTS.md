# MINGGU_8_UTS.md
**Topik:** Ujian Tengah Semester (UTS) – 3 Scenario Praktik + Troubleshooting  
**Tema Besar:** Integrasi Enterprise Network Head Office + Branch (Minggu 1-7) [cite:8][cite:18]

---

## 1. Instruksi UTS

**Durasi:** 120 menit (2x60 menit)  
**Bentuk:** **Hands-on praktik** di lab (tidak ada teori tertulis).  
**Kelompok:** 2-3 orang, 1 set perangkat per kelompok.  
**Penilaian:** 100 poin (30% nilai akhir mata kuliah). [cite:8][cite:18]  

**Tujuan UTS:** Mahasiswa mampu mengintegrasikan dan troubleshoot infrastruktur enterprise network yang sudah dibangun Minggu 1-7.  

---

## 2. Perangkat & Environment Per Kelompok

Sama seperti praktikum biasa: [cite:11][cite:12]  
- **Head Office:** RB1100-CORE, CRS326, srv-ho1 (semua services Minggu 1-7 aktif).  
- **Branch-1:** RB3011-BR1, PC-BR1.  
- **Akses:** SSH ke srv-ho1, Winbox ke MikroTik, browser di PC-BR1.  

**Status awal (disediakan dosen/asisten):**  
- Backbone connectivity OK (ping end-to-end).  
- BIND9, DHCP, Nginx, NFS/Samba, Prometheus/Grafana running.  
- **3 FAULTS tersembunyi** untuk ditemukan dan diperbaiki (lihat Scenario).  

---

## 3. Scenario UTS (Urut, Waktu 40 Menit Per Scenario)

### Scenario 1: DNS & DHCP Integration Failure (40 Menit, 35 Poin)

**Gejala yang diberikan:**  
```
PC-BR1 dapat IP 192.168.11.10 OK via DHCP RB3011
Tapi: nslookup srv-ho1.corp.pens.lab → SERVFAIL
ping 10.252.108.254 OK, tapi ping srv-ho1.corp.pens.lab → Name/Error
Grafana dashboard (https://portal.corp.pens.lab/grafana/) → DNS resolution fail
```

**Tugas:**  
1. Identifikasi dan perbaiki masalah **DHCP → DNS integration** (DHCP relay?).  
2. Pastikan PC-BR1 resolve semua hostname BIND9 (portal.corp.pens.lab, grafana.corp.pens.lab).  
3. Akses Grafana dashboard via Nginx proxy dan screenshot **CPU srv-ho1** dari Minggu 7.  
4. Dokumentasikan root cause dan solusi (3 kalimat).  

**Fault tersembunyi:** DHCP relay di RB3011 tidak forward DNS option atau BIND9 zone reload gagal. [cite:1][cite:6]  

**Checklist asisten (35 poin):**  
- [10] DHCP relay fixed, PC dapat DNS=10.252.108.21  
- [10] BIND9 resolve semua internal domains  
- [10] Grafana dashboard accessible + screenshot CPU  
- [5] Dokumentasi root cause jelas  

---

### Scenario 2: Web Services & Load Balancing Failure (40 Menit, 35 Poin)

**Gejala yang diberikan:**  
```
https://portal.corp.pens.lab → 502 Bad Gateway terus-menerus
curl http://10.252.108.22:8080 OK, tapi 10.252.108.23:8080 → Connection refused
Nginx status OK, SSL certificate valid
Grafana dashboard menunjukkan nginx uptime OK tapi response time >5s
```

**Tugas:**  
1. Identifikasi dan perbaiki **load balancing failure** di Nginx.  
2. Jalankan **20 request** ke portal.corp.pens.lab, pastikan distribusi traffic app1:app2 (sesuai weight).  
3. Simulasikan **failover**: matikan app1, verifikasi semua traffic ke app2.  
4. Screenshot **Grafana nginx response time** sebelum/sesudah fix.  
5. Dokumentasikan Nginx config yang diubah.  

**Fault tersembunyi:** Backend app2 down (Apache tidak running port 8080) + health_check Nginx timeout.  

**Checklist asisten (35 poin):**  
- [10] Kedua backend accessible port 8080  
- [10] Load balancing distribusi benar (screenshot 20 curl)  
- [10] Failover test sukses  
- [5] Grafana screenshot response time improvement  

---

### Scenario 3: File Services & Backup + Monitoring Alert (40 Menit, 30 Poin)

**Gejala yang diberikan:**  
```
mount //10.252.108.21/users → Permission denied
rsync dari PC-BR1 ke /backup/branch1 → Permission denied SSH
Grafana dashboard → Disk usage srv-ho1 >90%, backup folder penuh
NFS /srv/nfs/shared → read-only dari branch
Prometheus alert: node_filesystem_avail_bytes <10%
```

**Tugas:**  
1. Perbaiki **permission Samba/NFS/SSH** untuk akses dari branch.  
2. Jalankan **rsync backup** manual dari PC-BR1 ke srv-ho1 (backup /etc/).  
3. **Cleanup disk space** di /backup/ agar <70% usage.  
4. Screenshot **Grafana disk usage** sebelum/sesudah + Prometheus alert resolved.  
5. Setup **cronjob rsync** otomatis (setiap 2 jam) dan test.  

**Fault tersembunyi:** Permission /srv/samba/users salah, SSH key tidak setup, /backup penuh dengan old files.  

**Checklist asisten (30 poin):**  
- [8] Samba/NFS mount sukses dari PC-BR1 (read/write)  
- [8] rsync backup manual sukses + cronjob setup  
- [8] Disk usage <70%, Grafana screenshot improvement  
- [6] Prometheus alert resolved  

---

## 4. Instruksi Pengumpulan Hasil UTS

**Kirim via Google Drive kelompok (15 menit terakhir):** [cite:8]  

**File ZIP wajib berisi:**  
1. **Screenshot 9 gambar** (3 per scenario): before/after + result.  
2. **Dokumentasi text** (max 500 kata total):  
   - Root cause tiap scenario.  
   - Perintah/config yang diubah.  
   - Screenshot Grafana metrics improvement.  
3. **Grafana dashboard JSON export** (scenario 2 & 3).  

**Nama file:** `UTS-K[NomorKelompok]-Nim1-Nim2-Nim3.zip`

---

## 5. Kriteria Penilaian Detail

| Kriteria | Poin |  
|----------|------|  
| **Waktu selesai** (semua scenario <120 menit) | 10 |  
| Scenario 1 (DNS/DHCP) | 35 |  
| Scenario 2 (Web/Load Balancer) | 35 |  
| Scenario 3 (File/Monitoring) | 30 |  
| Dokumentasi + Screenshot lengkap | 20 |  
| **Total** | **130** (scale ke 100) |  

**Bonus (+10 poin):**  
- Semua Grafana panels custom dan insightful.  
- Dokumentasi dengan diagram Mermaid singkat.  
- Root cause analysis dengan log evidence.  

---

## 6. Petunjuk Troubleshooting Umum

```
# Cek service status
systemctl status bind9 isc-dhcp-server nginx apache2 nfs-kernel-server smbd prometheus grafana-server

# Cek log
journalctl -u nginx -f
tail -f /var/log/prometheus/prometheus.log

# Network connectivity
nc -zv 10.252.108.22 8080
ss -tuln | grep 8080

# Disk/Memory
df -h
free -h
```

**Dilarang:**  
- Restart server fisik.  
- Hapus file config tanpa backup.  
- Akses perangkat kelompok lain.  

---

## 7. Persiapan Sebelum UTS (Untuk Mahasiswa)

**Review cepat:**  
1. **Minggu 1-2:** Backbone connectivity, Linux networking.  
2. **Minggu 3-4:** BIND9 DNS, ISC DHCP relay.  
3. **Minggu 5:** Nginx reverse proxy + load balancing.  
4. **Minggu 6:** NFS/Samba/rsync permissions.  
5. **Minggu 7:** Prometheus/Grafana troubleshooting.  

**Perintah must-know:**  
```
dig @10.252.108.21 hostname
ip dhcp-server network print (MikroTik)
nginx -t && systemctl reload nginx
journalctl -u service -f
systemctl restart service
```

---

## 8. Instruksi Asisten/Dosen

1. **Setup fault** 30 menit sebelum UTS:  
   - Scenario 1: disable DHCP relay option DNS di RB3011.  
   - Scenario 2: stop Apache di app2 (10.252.108.23:8080).  
   - Scenario 3: chmod 700 /srv/samba/users, isi /backup dengan dummy files 2GB.  

2. **Distribusi kelompok:** 1 set perangkat per 3 orang.  
3. **Monitoring:** walk-around, jawab pertanyaan "kenapa begini?" tapi tidak kasih solusi langsung.  
4. **Pengumpulan:** verifikasi ZIP via shared drive, nilai dalam 24 jam.  

**Selamat UTS! Teori praktik Minggu 1-7 diuji di sini. 
---

