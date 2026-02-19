# workshop-admin-jar-2026-B

Ide utamanya: jadikan lab ini “mini enterprise” dengan satu **super network** yang konsisten dari Minggu 1–14: backbone, head office (server farm, monitoring, identity), dan beberapa branch (RB3011 + PC) sebagai kantor cabang. [perplexity]

## 1. Pemetaan Perangkat ke Enterprise Network 

- CRS326 24-port: core/distribution switch di backbone (VLAN, trunk ke semua router/PC). 
- RB1100 (2 unit): 
  - RB1100-CORE: head office router + gateway ke internet + NAT + firewall pusat. 
  - RB1100-BRANCH: router “kantor cabang besar” (bisa dipakai untuk skenario SD-WAN/WireGuard). 
- RB3011 (10 unit): 10 branch office router (1 router = 1 cabang). 
- D-Link manageable 24-port: access switch LAN untuk salah satu branch besar atau untuk mengumpulkan 12 PC ke backbone.
  - 2–3 PC jadi “server head office”: Proxmox, monitoring, identity (FreeIPA/Keycloak), main DNS/Web. 
  - Sisa PC: client di masing-masing branch (user kantor), sebagian bisa jalankan Docker/K8s node. 

Skema IP yang konsisten:
- Backbone/supernet: 10.252.108.0/24 (atau 10.0.0.0/8 seperti yang sudah kita pakai sebelumnya). 
- Head office server VLAN: 10.252.10.0/24 (Proxmox, DNS, Monitoring, Identity). 
- Branch: 10 cabang, misalnya 192.168.1X.0/24, X = 1..10 (satu subnet per RB3011). 
## 2. Pemetaan Topologi ke Urutan Minggu

Kuncinya: semua minggu pakai satu narasi “enterprise HQ + 10 branch” supaya mahasiswa merasa bangun sistem yang sama, makin lama makin lengkap. 
### Minggu 1 – Setup Environment Lab (Design Jaringan)

Fokus: bangun tulang punggung enterprise.

- Tujuan:
  - CRS326 sebagai core, RB1100-CORE sebagai gateway, backbone 10.252.108.0/24 online.
  - Minimal 1 RB3011 + 1 PC per kelompok terhubung sebagai “Branch-1”.
  - IP addressing dan skema VLAN di CRS326 (VLAN backbone, server, branch). [forum.mikrotik](https://forum.mikrotik.com/t/using-routeros-to-vlan-your-network/126489)
  - Install Proxmox di 1–2 PC → VM template Ubuntu server. [reddit](https://www.reddit.com/r/homelab/comments/1fcui9f/fully_functional_k8s_on_proxmox_using_terraform/)
  - Akses SSH dari PC client ke VM di server farm lewat RB1100-CORE.

File: MINGGU_1_PRAKTIKUM_LENGKAP.md fokus pada gambar topologi (Mermaid) dan mapping port fisik CRS326/RB1100/RB3011. [perplexity](https://www.perplexity.ai/search/79569a26-30c4-4ce2-a797-b778cfb40f5c)

### Minggu 2 – Linux Network Admin

Fokus: Linux di server farm & branch server.

- VM di head office: srv-core-1, srv-core-2. 
- VM di branch (di Proxmox branch atau langsung di PC): srv-branch-1. 
- Praktik:
  - Multi-IP dan alias untuk server multi-role (web + DNS + monitoring) di VLAN server. 
  - Bonding/bridge antar interface di server yang punya lebih dari satu NIC; simulasi link server ke dua switch. [dev](https://dev.to/patimapoochai/building-a-declarative-home-lab-using-k3s-ansible-helm-on-nixos-and-rocky-linux-21l5)
  - VLAN tagging di Linux untuk koneksi trunk ke CRS326 (server “langsung ke core”). [reddit](https://www.reddit.com/r/mikrotik/comments/169bmb1/guidance_for_vlan_routing_rb4011crs326/)

### Minggu 3 – DNS Services

Narasi: head office punya DNS pusat, branch punya caching forwarder.

- srv-dns-core (di HQ): BIND9 authoritative untuk zone corp.lab (proxmox.corp.lab, grafana.corp.lab, id.corp.lab). 
- srv-branch-X: dnsmasq atau BIND caching/forwarder yang forward ke DNS HQ. 
  - Forward + reverse zone, NS record antar site. 
  - Test resolusi nama dari PC di branch ke service di HQ.

### Minggu 4 – DHCP & NTP

Narasi: HQ sebagai core infra, branch punya DHCP lokal.

- RB3011 branch: DHCP server untuk subnet 192.168.1X.0/24. 
- srv-dhcp-core (optional) untuk server farm VLAN. 
- NTP/Chrony di srv-ntp-core, semua router dan server sync ke dia. 
- Latihan: relay vs DHCP lokal, pengaturan lease, dan clock konsisten untuk log & monitoring.

### Minggu 5 – Web Services & Proxy

Narasi: HQ meng-host aplikasi enterprise.

- srv-web-core: Nginx reverse proxy di depan beberapa VM (app1, app2). 
- SSL: self-signed atau internal CA dari identity server nanti. [dev](https://dev.to/patimapoochai/building-a-declarative-home-lab-using-k3s-ansible-helm-on-nixos-and-rocky-linux-21l5)
- Branch PC akses web: https://portal.corp.lab lewat RB1100-CORE.
- Tambahan: caching proxy di salah satu branch untuk efisiensi link.

### Minggu 6 – File Services & Storage

Narasi: file server pusat untuk semua branch.

- srv-file-core: Samba + NFS di server farm VLAN. 
- Mount dari branch: PC user map network drive ke \\file.corp.lab\share.
- rsync backup dari branch server (srv-branch-X) ke HQ file server via cron.

### Minggu 7 – Monitoring & Observability

Narasi: NOC di head office memonitor semua router dan server.

- srv-mon-core: Prometheus + Grafana di HQ. 
- node_exporter di:
  - Semua VM server.
  - Bisa juga Mikrotik via SNMP exporter. 
  - Health branch (ping loss, CPU RB3011 per cabang).
  - Health server farm (RAM/CPU/disk). [dev](https://dev.to/patimapoochai/building-a-declarative-home-lab-using-k3s-ansible-helm-on-nixos-and-rocky-linux-21l5)

### Minggu 8 – UTS

- Skenario integrasi: 
  - Satu branch down, harus restore konektivitas dan DNS/DHCP. 
  - Misroute NAT di RB1100-CORE, mahasiswa betulkan.
- 3 skenario berbeda untuk 3 kelompok seperti yang sudah Anda desain. 

### Minggu 9 – Container Networking

Narasi: sebagian aplikasi di HQ mulai di-container-kan.

- srv-container-core: Docker host di salah satu PC/VM HQ. [reddit](https://www.reddit.com/r/homelab/comments/1fcui9f/fully_functional_k8s_on_proxmox_using_terraform/)
- Praktik:
  - docker bridge vs host vs macvlan untuk koneksi ke VLAN server / branch. [reddit](https://www.reddit.com/r/homelab/comments/1fcui9f/fully_functional_k8s_on_proxmox_using_terraform/)
  - Compose untuk deploy layanan web + DB yang diakses dari branch.

### Minggu 10 – Kubernetes Networking

Narasi: HQ punya private K3s cluster sebagai platform aplikasi.

- 1 PC/VM sebagai K3s server, 2–3 VM/PC sebagai agent. 
- CNI Calico:
  - Mahasiswa lihat perbedaan antara pod CIDR, service CIDR, dan jaringan fisik. 
- NetworkPolicy: batasi aplikasi tertentu hanya dapat diakses dari subnet tertentu (misal hanya dari HQ VLAN). 

### Minggu 11 – Network Automation

Narasi: NOC mengotomasi konfigurasi router & server.

- Control node Ansible di HQ (srv-ansible-core). 
- Inventory berisi:
  - 2 RB1100 + 10 RB3011 (via SSH API).
  - Beberapa Linux server (DNS, web, monitoring).
- Praktik:
  - Playbook untuk push konfigurasi firewall dasar/nat ke semua branch.
  - Template untuk /etc/resolv.conf dan time sync.

### Minggu 12 – SDN/SD-WAN (WireGuard)

Narasi: SD-WAN untuk branch via overlay (WireGuard).

- RB1100-CORE sebagai hub WG.
- Beberapa RB3011 sebagai spoke (branch remote via WG overlay), walaupun fisiknya satu lab. 
- Praktik:
  - Setup WG interface di router Linux (bisa via VM) dan/atau Mikrotik (kalau firmware mendukung atau disimulasikan di Linux). [media.frnog](https://media.frnog.org/FRnOG_37/FRnOG_37-3.pdf)
  - Policy routing: traffic tertentu via overlay, lainnya direct internet.

### Minggu 13 – Network Security

Narasi: perkuat keamanan enterprise.

- nftables di Linux server (web, DNS) untuk segmentasi port. 
- Suricata IDS di HQ VLAN server untuk monitor trafik ke/dari branch. 
- fail2ban untuk melindungi SSH dan web login di HQ server.
- Latihan: buat rule drop tertentu, lihat alert Suricata ketika ada “serangan” dari PC branch.

### Minggu 14 – Project Presentation

- Setiap kelompok present:
  - Design branch mereka (IP, layanan yang di-host di branch).
  - Integrasi ke HQ (DNS, monitoring, security).
  - Demo real: akses aplikasi dari user branch, lihat Grafana, lihat log Suricata, dsb. 

## 3. Ide Konkrit Pembagian Per Perangkat

Contoh mapping cepat yang konsisten dengan di atas: 

- CRS326:
  - Port 1: ke RB1100-CORE.
  - Port 2: ke RB1100-BRANCH.
  - Port 3–12: ke 10 RB3011 (Branch1–Branch10).
  - Port 13–24: ke PC/Server farm (Proxmox, monitoring, DNS, web).
- D-Link 24-port:
  - Jadi access switch untuk 12 PC client + 2 server tambahan (jika butuh).
- IP backbone:
  - 10.252.108.254: RB1100-CORE.
  - 10.252.108.10–19: RB3011 (wan/backbone).
  - 10.252.108.21–26: PC/VM server farm.

