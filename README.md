# Panduan Lengkap: Mengatasi Disk Space Penuh pada 0G Storage Node
# og storage disk cleanup guide

## 🚨 **MISAL DATA NYA SEKITAR 317 GB**
- Server mengalami disk space penuh (85-90% usage)
- 0G Storage Node memakan space besar (~371GB di folder data_db)

## 🔍 **DIAGNOSIS MASALAH**

### 1. Cek Penggunaan Disk
```bash
# Cek overall disk usage
df -h

# Cek ukuran folder 0G Storage Node
du -sh /root/0g-storage-node/run/db/*

# Hasil yang ditemukan misal:
# 371G /root/0g-storage-node/run/db/data_db
# 317M /root/0g-storage-node/run/db/flow_db
```

### 2. Identifikasi Folder Besar
```bash
# Cek folder mana yang paling besar
du -sh /root/* | sort -hr

# Cek struktur database 0G
ls -la /root/0g-storage-node/run/db/
```

## ✅ **KONFIRMASI RESMI DARI TIM 0G**

**JAWABAN RESMI TIM 0G:**
- ✅ **BOLEH DIHAPUS:** `data_db` folder (371GB)
- ❌ **JANGAN HAPUS:** `flow_db` folder (317MB)
- ⚠️ **PERINGATAN:** Hanya hapus data_db, jaga flow_db untuk menghindari mining untuk orang lain

## 🛠️ **SOLUSI LENGKAP**

### LANGKAH 1: Persiapan dan Backup

```bash
# 1. Cek status node yang sedang berjalan
ps aux | grep zgs
ps aux | grep 0g-storage

# 2. Backup flow_db untuk jaga-jaga (PENTING!)
cp -r /root/0g-storage-node/run/db/flow_db /root/flow_db_backup

# 3. Verifikasi backup berhasil
ls -la /root/flow_db_backup/
```

### LANGKAH 2: Stop Service Dengan Aman

```bash
# 1. Stop 0G Storage Node service
sudo systemctl stop zgs

# 2. Konfirmasi service sudah stop
sudo systemctl status zgs
# Status harus menunjukkan: "inactive (dead)"

# 3. Pastikan tidak ada process yang masih berjalan
ps aux | grep zgs
ps aux | grep 0g-storage
# Hanya boleh ada grep process saja
```

### LANGKAH 3: Verifikasi Sebelum Penghapusan

```bash
# 1. Cek struktur folder sekali lagi
ls -la /root/0g-storage-node/run/db/

# 2. Konfirmasi ukuran folder
du -sh /root/0g-storage-node/run/db/data_db
du -sh /root/0g-storage-node/run/db/flow_db

# 3. Pastikan hanya ada 2 folder: data_db dan flow_db
```

### LANGKAH 4: Penghapusan Folder data_db

```bash
# EKSEKUSI PENGHAPUSAN (HATI-HATI!)
rm -rf /root/0g-storage-node/run/db/data_db

# JANGAN SAMPAI SALAH KETIK!
# Yang dihapus: data_db (371GB)
# Yang TIDAK boleh dihapus: flow_db (317MB)
```

### LANGKAH 5: Verifikasi Penghapusan

```bash
# 1. Cek apakah data_db sudah terhapus
ls -la /root/0g-storage-node/run/db/
# Harus hanya ada flow_db saja

# 2. Cek disk space yang sudah kosong
df -h
# Available space harus bertambah ~371GB

# 3. Pastikan flow_db masih ada dan utuh
du -sh /root/0g-storage-node/run/db/flow_db
```

### LANGKAH 6: Restart Service

```bash
# 1. Start service kembali
sudo systemctl start zgs

# 2. Cek status service
sudo systemctl status zgs
# Status harus: "active (running)"

# 3. Verifikasi node berjalan normal
ps aux | grep zgs
```

## 📊 **HASIL YANG DIHARAPKAN**

### Sebelum Penghapusan:
- Disk Usage: 85-90% penuh
- Available Space: ~20GB
- data_db: 371GB
- flow_db: 317MB

### Setelah Penghapusan:
- Disk Usage: 5-10% 
- Available Space: ~353GB
- data_db: Tidak ada (akan terbentuk kembali)
- flow_db: 317MB (tetap utuh)

## 🔄 **PROSES SETELAH PENGHAPUSAN**

### Yang Akan Terjadi:
1. **Node akan re-sync data** secara otomatis dari network
2. **data_db akan terbentuk kembali** secara bertahap
3. **flow_db tetap utuh** sehingga tidak perlu re-mining
4. **Proses re-sync memakan waktu** tergantung kecepatan network

### Monitoring Pasca-Penghapusan:
```bash
# Monitor pertumbuhan data_db baru
watch -n 60 "du -sh /root/0g-storage-node/run/db/* 2>/dev/null"

# Cek log node
tail -f /root/0g-storage-node/run/log

# Monitor disk space
df -h
```

## ⚠️ **PERINGATAN PENTING**

### YANG BOLEH DIHAPUS misalnya:
- ✅ `data_db` folder (371GB)

### YANG JANGAN DIHAPUS:
- ❌ `flow_db` folder (317MB)
- ❌ File konfigurasi node
- ❌ Binary executable node

### TIPS KEAMANAN:
1. **SELALU backup flow_db** sebelum melakukan apapun
2. **STOP service dulu** sebelum menghapus folder
3. **VERIFIKASI nama folder** sebelum rm -rf
4. **COBA di testnet dulu** jika memungkinkan

Prosedur ini dikonfirmasi langsung oleh **Tim 0G Official** dengan pertanyaan:
- **Q:** "Can I delete some file or what to do?"
- **A:** "You can delete the data_db folder to free up space. Ensure you keep the flow_db folder and only delete the data_db folder to avoid mining for others"

**Status:** ✅ **OFFICIALLY CONFIRMED** oleh Tim 0G
