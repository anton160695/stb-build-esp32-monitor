# 🖥️ STB Hardware Monitor - Pre-built Binaries

Repositori ini menyediakan biner siap pakai (*pre-compiled binaries*) dari aplikasi **STB Hardware Monitor**. Program ini bertugas membaca metrik hardware STB (suhu SoC/CPU, RAM, storage, uptime, Docker container) dan mengirimkannya ke layar monitor ESP32.

---

## ⚡ Cara Cepat Pasang di STB (1 Perintah Selesai)

Karena repository ini bersifat **public**, Anda bisa langsung login ke STB via SSH dan menjalankan salah satu perintah di bawah ini sesuai jenis STB Anda. Perintah ini otomatis mendownload, menamai file menjadi `stb_monitor`, dan memberikan izin eksekusi (`chmod +x`):

### 🔹 Untuk STB 64-bit (Amlogic S905X/X2/X3/X4, Allwinner H6, RK3328, Raspberry Pi 4/5):
```bash
curl -L -o /usr/local/bin/stb_monitor https://raw.githubusercontent.com/anton160695/stb-build-esp32-monitor/main/stb_monitor_arm64 && chmod +x /usr/local/bin/stb_monitor
```

### 🔹 Untuk STB 32-bit (ZTE B860H V1/V2, Fiberhome HG680P):
```bash
curl -L -o /usr/local/bin/stb_monitor https://raw.githubusercontent.com/anton160695/stb-build-esp32-monitor/main/stb_monitor_arm7 && chmod +x /usr/local/bin/stb_monitor
```

### 🔹 Untuk PC / Mini PC / Home Server (x86_64 / Intel / AMD):
```bash
curl -L -o /usr/local/bin/stb_monitor https://raw.githubusercontent.com/anton160695/stb-build-esp32-monitor/main/stb_monitor_amd64 && chmod +x /usr/local/bin/stb_monitor
```

> **Tips Jika Menggunakan `wget` (Bila `curl` belum terpasang):**
> ```bash
> wget -O /usr/local/bin/stb_monitor https://raw.githubusercontent.com/anton160695/stb-build-esp32-monitor/main/stb_monitor_arm64 && chmod +x /usr/local/bin/stb_monitor
> ```

---

## 🤖 Pilihan: Script Auto-Detect (Download Otomatis Sesuai Arsitektur)

Jika Anda tidak yakin arsitektur STB Anda, cukup copy-paste blok perintah ini langsung ke terminal STB:

```bash
ARCH=$(uname -m)
if [ "$ARCH" = "aarch64" ]; then
    BIN="stb_monitor_arm64"
elif [ "$ARCH" = "armv7l" ]; then
    BIN="stb_monitor_arm7"
else
    BIN="stb_monitor_amd64"
fi
echo "Mendownload $BIN untuk arsitektur $ARCH..."
curl -L -o /usr/local/bin/stb_monitor "https://raw.githubusercontent.com/anton160695/stb-build-esp32-monitor/main/$BIN" && chmod +x /usr/local/bin/stb_monitor && echo "Selesai! Biner terpasang di /usr/local/bin/stb_monitor"
```

---

## 📁 Daftar File Biner

| Nama File | Arsitektur | Target Perangkat / STB |
| :--- | :--- | :--- |
| **`stb_monitor_arm7`** | ARM 32-bit (`armv7l`) | STB Armbian 32-bit (ZTE B860H V1/V2, Fiberhome HG680P) |
| **`stb_monitor_arm64`** | ARM 64-bit (`aarch64`) | STB Armbian 64-bit (Amlogic S905X Series, Allwinner H6, RK3328, Pi 4/5) |
| **`stb_monitor_amd64`** | x86_64 (64-bit PC) | Mini PC, Komputer Desktop, VPS, Home Server (Intel/AMD) |

---

## 🧪 Uji Coba Manual

Setelah terpasang, jalankan perintah ini di STB untuk mengetes apakah aplikasi berjalan normal:

```bash
/usr/local/bin/stb_monitor
```
*(Tekan `Ctrl + C` untuk menghentikan pengujian)*

---

## 🔄 Pasang Auto-Start (Systemd Service)

Agar aplikasi otomatis berjalan di background saat STB booting atau menyala kembali:

1. **Buat file service:**
   ```bash
   cat << 'EOF' > /etc/systemd/system/stb-monitor.service
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
   EOF
   ```

2. **Aktifkan & Jalankan Service:**
   ```bash
   systemctl daemon-reload
   systemctl enable stb-monitor
   systemctl start stb-monitor
   ```

3. **Cek Status & Pantau Log:**
   ```bash
   # Cek apakah service aktif (Active: running)
   systemctl status stb-monitor

   # Melihat log pengiriman data secara real-time
   journalctl -u stb-monitor -f
   ```

---

## 🛑 Perintah Pengelolaan Service

| Perintah | Fungsi |
| :--- | :--- |
| `systemctl status stb-monitor` | Melihat status service |
| `systemctl restart stb-monitor` | Merestart service |
| `systemctl stop stb-monitor` | Menghentikan service |
| `systemctl disable stb-monitor` | Menonaktifkan auto-start |
| `journalctl -u stb-monitor -n 50 --no-pager` | Melihat 50 baris log terakhir |