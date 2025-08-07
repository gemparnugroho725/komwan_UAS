Berikut langkah lengkap dari **build aplikasi** sampai **Horizontal Pod Autoscaler (HPA)** tanpa memakai Docker Hub/username, untuk cluster lokal seperti di WSL:

---

### 1. Build Docker Image

```bash
docker build -t cryptixguard:latest .
```

---

### 2. Buat file Deployment (deployment.yaml) dan Service (service.yaml) yang memakai image `cryptixguard:latest` tanpa username/registry.

---

### 3. Terapkan Deployment di Kubernetes

```bash
kubectl apply -f k8s/deployment.yaml
```

---

### 4. Terapkan Service untuk expose aplikasi

```bash
kubectl apply -f k8s/service.yaml
```

---

### 5. Cek pod dan service berjalan

```bash
kubectl get pods
kubectl get svc
```

---

### 6. Buat file Horizontal Pod Autoscaler (hpa.yaml) yang mengacu ke Deployment `cryptixguard`.

---

### 7. Terapkan HPA

```bash
kubectl apply -f k8s/hpa.yaml
```

---

### 8. Cek status HPA

```bash
kubectl get hpa
kubectl describe hpa cryptixguard-hpa
```

---

### 9. Akses aplikasi melalui NodePort/LoadBalancer IP sesuai service

---

Kalau kamu pakai Docker Desktop di WSL, image lokal biasanya langsung bisa dipakai Kubernetes tanpa push ke registry.

Kalau ingin saya bantu buatkan contoh YAML atau cek konfigurasi, tinggal bilang.
