Oke, dari info OpenShift Web Console yang kamu kirim, berikut poin penting yang bisa kita simpulkan:

---

### 🔍 Detail Aplikasi:

* **Nama Deployment:** `encryption-api`
* **Namespace (Project):** `gemparnugroho725-dev`
* **Image Source:** Dari internal registry OpenShift

  ```
  image-registry.openshift-image-registry.svc:5000/gemparnugroho725-dev/encryption-api
  ```
* **Pod:** 1 pod aktif
* **Belum ada HPA / autoscaler yang terpasang**

---

### ✅ Langkah selanjutnya: Pasang dan uji **HorizontalPodAutoscaler (HPA)**

#### 1. **Contoh YAML HPA untuk `encryption-api`**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: encryption-api-hpa
  namespace: gemparnugroho725-dev
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: encryption-api
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 20
```

#### 2. **Apply HPA di terminal OpenShift Web Console**

Salin YAML di atas ke file `hpa-encryption-api.yaml`, lalu di terminal OpenShift kamu:

```bash
oc apply -f hpa-encryption-api.yaml
```

---

### 🔥 Cara uji autoscaling

#### Jalankan load ke aplikasi:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: load-generator
  namespace: gemparnugroho725-dev
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "while true; do wget -q -O- http://encryption-api:5000/; done"]
```

Apply di terminal:

```bash
oc apply -f load-generator.yaml
```

---

### 👀 Pantau hasilnya

```bash
oc get hpa -n gemparnugroho725-dev
oc get pods -n gemparnugroho725-dev -w
```

Kalau HPA aktif dan load cukup tinggi, jumlah pod dari `encryption-api` akan bertambah otomatis.

---

Kalau kamu mau YAML HPA dan load generator aku buatin jadi satu file atau paketkan sekalian, tinggal bilang.

Untuk **mematikan (menghapus) HPA dan menghentikan load-generator** di OpenShift, cukup lakukan dua langkah:

---

### 🛑 1. **Hapus HPA**

Jika kamu sudah buat HPA dengan nama `encryption-api-hpa`:

```bash
oc delete hpa encryption-api-hpa -n gemparnugroho725-dev
```

---

### 🛑 2. **Hentikan load-generator**

Kalau kamu jalankan `load-generator` dengan nama pod `load-generator`, hapus pod-nya:

```bash
oc delete pod load-generator -n gemparnugroho725-dev
```

---

### ✅ (Opsional) Kembalikan jumlah pod manual (jika masih lebih dari 1)

Jika HPA sudah men-scale `encryption-api` menjadi lebih dari 1 pod, dan kamu ingin mengembalikannya ke 1 pod:

```bash
oc scale deployment encryption-api --replicas=1 -n gemparnugroho725-dev
```

---

Setelah itu, kamu bisa cek semua bersih:

```bash
oc get hpa -n gemparnugroho725-dev
oc get pods -n gemparnugroho725-dev
```

Kalau masih ada yang nyangkut, kabari saja.

