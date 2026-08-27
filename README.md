# 🖥️ STB Hardware Monitor - Pre-built Binaries

Repositori ini menyediakan biner siap pakai (*pre-compiled binaries*) dari aplikasi **STB Hardware Monitor** yang ditulis dalam Go. Program ini berfungsi memonitor status perangkat (suhu CPU/SoC, pemakaian RAM/Storage, uptime, status container Docker, dsb.) dan mempublikasikan datanya (misal via MQTT/Serial) untuk ditampilkan di layar monitor ESP32.

---

## 📁 Daftar File Biner

Pilihlah file biner yang sesuai dengan arsitektur STB atau komputer Anda:

| Nama File | Arsitektur | Target Perangkat / STB |
| :--- | :--- | :--- |
| **`stb_monitor_arm7`** | ARM 32-bit (`armv7l`) | STB Armbian 32-bit (ZTE B860H V1/V2, Fiberhome HG680P, dsb.) |
| **`stb_monitor_arm64`** | ARM 64-bit (`aarch64`) | STB Armbian 64-bit (Amlogic S905X/X2/X3/X4, Allwinner H6/H616, RK3328/RK3399, Raspberry Pi 4/5) |
| **`stb_monitor_amd64`** | x86_64 (64-bit PC) | Mini PC, Komputer Desktop, VPS, atau Home Server (Intel/AMD) |

---

## 🔍 1. Cek Arsitektur STB Anda

Sebelum memasang, login ke STB via SSH dan jalankan perintah berikut untuk mengetahui arsitektur CPU STB:

```bash
uname -m
```

* **`armv7l`** 👉 Gunakan file **`stb_monitor_arm7`**
* **`aarch64`** 👉 Gunakan file **`stb_monitor_arm64`**
* **`x86_64`** 👉 Gunakan file **`stb_monitor_amd64`**

---

## 🚀 2. Cara Memasukkan File ke STB

Pilih salah satu metode transfer file di bawah ini yang paling mudah:

### Opsi A: Transfer Menggunakan SCP (Terminal PC / PowerShell)
Dari komputer lokal Anda, buka Command Prompt / PowerShell / Terminal, lalu jalankan:

```bash
# Contoh jika STB Anda menggunakan arsitektur 64-bit (arm64):
scp stb_monitor_arm64 root@<IP_STB>:/usr/local/bin/stb_monitor

# Atau jika STB Anda menggunakan arsitektur 32-bit (arm7):
scp stb_monitor_arm7 root@<IP_STB>:/usr/local/bin/stb_monitor
```
*(Ganti `<IP_STB>` dengan alamat IP STB Anda, contoh: `192.168.1.100`)*

---

### Opsi B: Transfer Menggunakan WinSCP / FileZilla (GUI Windows)
1. Buka aplikasi **WinSCP** atau **FileZilla**.
2. Masukkan Host: `IP_STB`, Username: `root`, Port: `22`, dan password STB Anda.
3. Masuk ke direktori `/usr/local/bin/` pada panel kanan (STB).
4. *Drag and drop* file biner yang sesuai dari PC Anda ke folder tersebut.
5. Rename file di STB menjadi `stb_monitor`.

---

### Opsi C: Download Langsung di Terminal STB via Wget / Curl
Jika file sudah di-upload ke GitHub Anda:

```bash
# Masuk sebagai root di STB
cd /usr/local/bin

# Download file (ganti USERNAME dan NAMA_FILE sesuai kebutuhan)
curl -L -o stb_monitor https://github.com/<USERNAME>/<REPO>/raw/main/stb_monitor_arm64
```

---

## ⚙️ 3. Cara Menjalankan Program

### Langkah 1: Berikan Hak Akses Eksekusi
Masuk ke STB via SSH, lalu berikan izin *executable*:

```bash
chmod +x /usr/local/bin/stb_monitor
```

---

### Langkah 2: Uji Coba Manual
Jalankan program secara langsung untuk memastikan pembacaan sensor dan monitoring berjalan lancar:

```bash
# Jalankan langsung
/usr/local/bin/stb_monitor

# Atau jalankan dalam mode dry-run/tes (jika tersedia opsi diagnostik)
/usr/local/bin/stb_monitor -dry-run
```
*Tekan `Ctrl + C` untuk menghentikan program jika sudah selesai diuji.*

---

### Langkah 3: Menjalankan Otomatis di Background (Systemd Service)

Agar program tetap berjalan di latar belakang dan otomatis aktif kembali saat STB di-restart / booting:

1. **Buat file service systemd:**
   ```bash
   nano /etc/systemd/system/stb-monitor.service
   ```

2. **Tempelkan (Paste) konfigurasi berikut:**
   ```ini
   [Unit]
   Description=STB Hardware Monitor Service for ESP32
   After=network.target docker.service
   Wants=docker.service

   [Service]
   Type=simple
   ExecStart=/usr/local/bin/stb_monitor
   Restart=always
   RestartSec=5s
   User=root

   [Install]
   WantedBy=multi-user.target
   ```
   *Simpan dengan menekan `Ctrl + O`, lalu `Enter`, kemudian keluar dengan `Ctrl + X`.*

3. **Aktifkan dan jalankan service:**
   ```bash
   # Muat ulang konfigurasi systemd
   systemctl daemon-reload

   # Aktifkan auto-start saat boot
   systemctl enable stb-monitor

   # Jalankan service sekarang
   systemctl start stb-monitor
   ```

4. **Cek status & log service:**
   ```bash
   # Cek status service
   systemctl status stb-monitor

   # Pantau log secara real-time
   journalctl -u stb-monitor -f
   ```

---

## 🛑 Perintah Pengelolaan Service

| Perintah | Fungsi |
| :--- | :--- |
| `systemctl status stb-monitor` | Melihat status apakah service berjalan atau error |
| `systemctl restart stb-monitor` | Merestart service |
| `systemctl stop stb-monitor` | Menghentikan service |
| `systemctl disable stb-monitor` | Menonaktifkan auto-start saat booting |
| `journalctl -u stb-monitor -n 50 --no-pager` | Melihat 50 baris log terakhir |