Berikut adalah **Rangkuman Komprehensif PaaS dengan Minishift dan OpenShift** versi lengkap yang sudah saya tambahkan **cara pengelolaan via GUI (Web Console)** selain **CMD/CLI**:

---

## **Rangkuman Komprehensif: PaaS dengan Minishift dan OpenShift**

Dokumen ini merangkum tiga modul pembelajaran mengenai penggunaan OpenShift sebagai Platform as a Service (PaaS) yang dijalankan secara lokal menggunakan Minishift. Rangkuman ini mencakup **konsep dasar, instalasi, pengelolaan user & proyek, build, deployment, scaling manual & otomatis baik via CLI maupun GUI (Web Console).**

---

### **1. Pengantar dan Instalasi**

| **Topik**                  | **Penjelasan**                                                                                                                                                                                                           |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Konsep Dasar OpenShift** | OpenShift adalah platform aplikasi kontainer berbasis Kubernetes untuk pengembangan, deployment, dan manajemen aplikasi.                                                                                                 |
| **Arsitektur**             | **Master Node** (API Server, etcd, Scheduler, Controller) & **Worker Node** (menjalankan aplikasi dalam Pod).                                                                                                            |
| **Minishift**              | Alat untuk menjalankan OpenShift single-node di VM secara lokal, ideal untuk pengembangan & pembelajaran.                                                                                                                |
| **Instalasi Minishift**    | Membutuhkan hypervisor seperti VirtualBox. Perintah: <br> **CLI** → `minishift start --vm-driver virtualbox` <br>**GUI** → Akses Web Console di `https://192.168.99.102:8443` <br> User: `developer`, Password: `bebas`. |

---

### **2. Pengelolaan User**

#### **A. Via CLI (Command Line Interface)**

| **Aksi**                       | **Perintah**                                                       |
| ------------------------------ | ------------------------------------------------------------------ |
| **Login Developer**            | `oc login -u developer -p developer`                               |
| **Login Admin Sistem**         | `oc login -u system:admin`                                         |
| **Melihat Daftar User**        | `oc get users`                                                     |
| **Memberikan Hak Akses Admin** | `oc adm policy add-cluster-role-to-user cluster-admin <nama-user>` |
| **Membuat User Baru (Admin)**  | `oc create user <nama-user>`                                       |
| **Menghapus User**             | `oc delete user <nama-user>`                                       |

#### **B. Via GUI (Web Console)**

| **Aksi**                    | **Langkah di GUI**                                                                                   |
| --------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Login ke Web Console**    | Buka URL Web Console → Masuk sebagai `system:admin` untuk akses penuh.                               |
| **Melihat Daftar User**     | Menu **Administration → Users** (akan tampil semua user).                                            |
| **Menambah User Baru**      | (Jika fitur diaktifkan) Klik **Create User → Masukkan nama user → Simpan**.                          |
| **Memberikan Role ke User** | Buka **Administration → Role Bindings → Assign Role to User** → Pilih role (misal: `edit`, `admin`). |
| **Menghapus User**          | Pilih user di menu **Users → Actions → Delete**.                                                     |

---

### **3. Pengelolaan Proyek**

#### **A. Via CLI**

| **Aksi**                          | **Perintah**                                                       |
| --------------------------------- | ------------------------------------------------------------------ |
| **Melihat Semua Proyek**          | `oc get projects`                                                  |
| **Membuat Proyek Baru**           | `oc new-project <nama-proyek> --display-name="Nama Tampilan"`      |
| **Mengganti Proyek Aktif**        | `oc project <nama-proyek>`                                         |
| **Menghapus Proyek**              | `oc delete project <nama-proyek>`                                  |
| **Memberikan Akses ke User Lain** | `oc adm policy add-role-to-user edit <nama-user> -n <nama-proyek>` |

#### **B. Via GUI**

| **Aksi**                          | **Langkah di GUI**                                                       |
| --------------------------------- | ------------------------------------------------------------------------ |
| **Melihat Semua Proyek**          | Menu **Home → Projects**.                                                |
| **Membuat Proyek Baru**           | Klik **Create Project → Isi nama proyek & display name → Create**.       |
| **Menghapus Proyek**              | Buka proyek → Klik **Actions → Delete Project**.                         |
| **Memberikan Akses ke User Lain** | Masuk ke proyek → **Project → Role Bindings → Add** → Pilih user & role. |

---

### **4. Build Aplikasi**

#### **A. Via CLI**

| **Aksi**                      | **Perintah**                                                            |
| ----------------------------- | ----------------------------------------------------------------------- |
| **Menjalankan Build Manual**  | `oc start-build <nama-build-config>`                                    |
| **Melihat Status Build**      | `oc get builds`                                                         |
| **Webhook Otomatis (GitLab)** | Tambahkan URL Webhook di pengaturan GitLab dengan URL dari BuildConfig. |

#### **B. Via GUI**

| **Aksi**                     | **Langkah di GUI**                                                       |
| ---------------------------- | ------------------------------------------------------------------------ |
| **Membuat Build**            | Menu **Builds → Create Build → Pilih Source-to-Image atau Dockerfile**.  |
| **Menjalankan Build Manual** | Pilih **Build → Actions → Start Build**.                                 |
| **Melihat Log Build**        | Klik **Build → Logs**.                                                   |
| **Webhook Otomatis**         | Buka **Build → Configuration → Webhook** → Salin URL & tempel di GitLab. |

---

### **5. Deployment Aplikasi**

#### **A. Via CLI**

| **Aksi**                          | **Perintah**                                    |
| --------------------------------- | ----------------------------------------------- |
| **Menjalankan Deployment Manual** | `oc rollout latest dc/<nama-deployment-config>` |
| **Melihat Log Deployment**        | `oc logs -f dc/<nama-deployment-config>`        |

#### **B. Via GUI**

| **Aksi**                          | **Langkah di GUI**                                                                       |
| --------------------------------- | ---------------------------------------------------------------------------------------- |
| **Menjalankan Deployment Manual** | Menu **Applications → Deployments → Actions → Start Rollout**.                           |
| **Melihat Log Deployment**        | Klik **Deployments → Pods → View Logs**.                                                 |
| **Strategi Deployment**           | Bisa diatur di **Deployment Config → Edit → Strategy (Rolling/Blue-Green/A-B Testing)**. |

---

### **6. Scaling Proyek (Manual & Otomatis)**

#### **A. Scaling Manual**

| **Metode** | **Langkah**                                                                                                                    |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **CLI**    | `oc scale dc/<nama-deployment-config> --replicas=<jumlah-replika>` <br>Contoh: `oc scale dc/simple-webapp-docker --replicas=5` |
| **GUI**    | Buka **Applications → Deployments → Scaling → Geser Slider / Edit jumlah replika → Save**                                      |

#### **B. Scaling Otomatis (HPA)**

| **Metode** | **Langkah**                                                                                                 |
| ---------- | ----------------------------------------------------------------------------------------------------------- |
| **CLI**    | `oc autoscale dc/<nama-deployment-config> --min=1 --max=10 --cpu-percent=80`                                |
| **GUI**    | **Applications → Deployments → Horizontal Pod Autoscaler → Create HPA → Isi min, max, & target CPU → Save** |

---

### **7. REST API**

| **Aksi**              | **Perintah**                                                                              |
| --------------------- | ----------------------------------------------------------------------------------------- |
| **Mendapatkan Token** | `oc whoami -t`                                                                            |
| **Akses via curl**    | `curl -k https://192.168.99.102:8443/oapi/v1/projects -H "Authorization: Bearer <TOKEN>"` |

---

✅ **Selanjutnya saya bisa bantu buatkan:**

1. **Versi dokumen siap cetak (format laporan akademik, rapi dengan numbering & daftar isi).**
2. **Infografis ringkas (visual untuk presentasi).**
3. **Langkah-langkah praktik (hands-on) step-by-step per modul.**

➡️ **Mau saya buatkan yang mana dulu?**
