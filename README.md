# Panduan Lengkap: Mengatasi Disk Space Penuh pada 0G Storage Node
# OG-storage disk cleanup guides

## 🚨 **MISAL DATA NYA SEKITAR 317 GB**
- Server mengalami disk space penuh (85-90% usage)
- 0G Storage Node memakan space besar (~371GB di folder data_db)

## 🔍 **DIAGNOSIS MASALAH**

### 1. Cek Penggunaan Disk memori
```basah
df -h
# Cek ukuran folder 0G Storage Node
du -sh /root/0g-storage-node/run/db/*

# Hasil yang ditemukan misal:
# 371G /root/0g-storage-node/run/db/data_db
# 317M /root/0g-storage-node/run/db/flow_db
```

### 2. Identifikasi Folder
```bash
# Cek folder mana yang paling besar
du -sh /root/* | sort -hr
ls -la /root/0g-storage-node/run/db/
```

## ✅ **KONFIRMASI PENTING**

**BAGUAN YANG BOLEH:**
- ✅ **BOLEH DIHAPUS:** `data_db` folder (371GB)
- ❌ **JANGAN HAPUS:** `flow_db` folder (317MB)
- ⚠️ **PERINGATAN:** Hanya hapus data_db, jaga flow_db untuk menghindari mining outbox

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
sudo systemctl status zgs
ps aux | grep zgs
ps aux | grep 0g-storage
```

### LANGKAH 3: Verifikasi Sebelum Penghapusan

```bash
# 1. Cek struktur folder sekali lagi
ls -la /root/0g-storage-node/run/db/

# 2. Konfirmasi ukuran folder
du -sh /root/0g-storage-node/run/db/data_db
du -sh /root/0g-storage-node/run/db/flow_db

# 3. Pastiin lagi hanya ada 2 folder: data_db dan flow_db
```

### LANGKAH 4: Penghapusan Folder data_db

```bash
rm -rf /root/0g-storage-node/run/db/data_db

# JANGAN SAMPAI SALAH KETIK!
# Yang dihapus: data_db (371GB) misal
# Yang TIDAK boleh dihapus: flow_db (317MB) misal
```

### LANGKAH 5: Verifikasi Penghapusan

```bash
ls -la /root/0g-storage-node/run/db/
# Harus hanya ada flow_db saja
# Pastikan flow_db masih ada dan utuh
du -sh /root/0g-storage-node/run/db/flow_db
```

### LANGKAH 6: Restart Service

```bash
sudo systemctl start zgs
sudo systemctl status zgs
ps aux | grep zgs
```

## 🔄 **PROSES SETELAH PENGHAPUSAN**

### Yang Akan Terjadi:
1. **Node akan re-sync data** secara otomatis dari network
2. **data_db akan terbentuk kembali** secara bertahap
3. **flow_db tetap utuh** sehingga tidak perlu re-mining
4. **Proses re-sync memakan waktu** tergantung kecepatan network

### Monitoring Pasca-Penghapusan:
```bash
watch -n 60 "du -sh /root/0g-storage-node/run/db/* 2>/dev/null"
# Cek log node
tail -f /root/0g-storage-node/run/log
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