Berikut adalah **step by step install Minishift 1.24 di Windows (VirtualBox sebagai hypervisor)** sampai OpenShift berjalan:

---

## ✅ **1. Persiapan & Kebutuhan Sistem**

| Komponen         | Keterangan                                          |
| ---------------- | --------------------------------------------------- |
| **OS**           | Windows 10/11 64-bit                                |
| **Virtualisasi** | Pastikan **Intel VT-x/AMD-V** aktif di BIOS         |
| **Hypervisor**   | VirtualBox (disarankan versi 6.1.x)                 |
| **RAM**          | Minimal 4 GB (untuk VM Minishift)                   |
| **CPU**          | Minimal 2 core                                      |
| **Disk**         | Minimal 20 GB kosong                                |
| **Internet**     | Stabil karena akan download ISO dan image OpenShift |

---

## ✅ **2. Download & Ekstraksi Minishift**

1. Download Minishift 1.24.0:
   👉 [https://github.com/minishift/minishift/releases/tag/v1.24.0](https://github.com/minishift/minishift/releases/tag/v1.24.0)
   Pilih file: **`minishift-1.24.0-windows-amd64.zip`**
2. Ekstrak ke folder, misalnya:

   ```
   C:\minishift-1.24.0-windows-amd64
   ```
3. Buka **Command Prompt (CMD)** atau **PowerShell**, masuk ke folder tersebut:

   ```cmd
   cd C:\minishift-1.24.0-windows-amd64
   ```

---

## ✅ **3. Jalankan Minishift Pertama Kali**

1. Jalankan perintah:

   ```cmd
   .\minishift.exe start --vm-driver virtualbox
   ```

2. Minishift akan otomatis:

   * Download **CentOS ISO** (sekitar 344 MB)
   * Download **oc binary** (OpenShift client)
   * Membuat VM di VirtualBox

3. Tunggu hingga selesai. Kalau berhasil akan muncul:

   ```
   OpenShift server started
   The server is accessible via web console at:
       https://192.168.xx.xx:8443
   ```

---

## ✅ **4. Cek Status Minishift**

```cmd
.\minishift.exe status
```

✅ **Hasil Normal**:

```
Minishift:  Running
Profile:    minishift
OpenShift:  Running
DiskUsage:  ...
```

Jika **OpenShift: Stopped**, berarti ada masalah (biasanya DNS/Internet dalam VM).

---

## ✅ **5. Perbaiki Internet di VM (Jika Error DNS)**

Kadang Minishift bisa ping `8.8.8.8` tapi tidak bisa resolve DNS (`ping google.com` gagal).
Cara perbaikinya:

### 🔹 **Masuk ke VM**

```cmd
.\minishift.exe ssh
```

### 🔹 **Edit DNS**

```bash
sudo vi /etc/resolv.conf
```

Ubah isinya jadi:

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

Lalu agar tidak kembali ke `10.0.2.3` saat restart:

```bash
sudo bash -c 'echo -e "[main]\ndns=none" > /etc/NetworkManager/conf.d/no-dns.conf'
sudo systemctl restart NetworkManager
```

Cek lagi:

```bash
ping google.com
```

✅ Kalau sudah reply, lanjut.

---

## ✅ **6. Restart Minishift/OpenShift**

Setelah DNS normal:

```cmd
.\minishift.exe stop
.\minishift.exe delete
.\minishift.exe start --vm-driver virtualbox
```

Tunggu hingga OpenShift Running.

---

## ✅ **7. Konfigurasi OC Client**

### 🔹 **Set `oc` ke PATH sementara**

```cmd
FOR /f "tokens=*" %i IN ('minishift oc-env') DO @call %i
```

Cek:

```cmd
oc version
```

### 🔹 **Login ke OpenShift**

```cmd
oc login -u system:admin
```

---

## ✅ **8. Akses Web Console**

Cek URL:

```cmd
.\minishift.exe console --url
```

Buka di browser (contoh):

```
https://192.168.99.105:8443
```

Login pakai:

* **Username**: `developer` (untuk normal user) atau `system:admin`
* **Password**: `any` (bebas, misalnya `admin`)

---

### ⚠ **Jika Masih Gagal OpenShift Running**

Biasanya masalahnya di DNS atau image OpenShift tidak berhasil ter-pull.
Kita bisa pull manual image lewat SSH.

---

❓ **Mau saya bikinin juga step khusus untuk:**

1. **Set PATH `oc` permanen di Windows**, atau
2. **Cara manual pull image OpenShift supaya OpenShift langsung Running?**
   Mau lanjut yang mana dulu?
