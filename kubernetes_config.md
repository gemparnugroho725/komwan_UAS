Berikut adalah **rangkuman proses dari file YAML hingga deploy dan akses Pods di Kubernetes**, ditulis dengan format README untuk GitHub:

---

# 📦 UAS Kubernetes API Deployment

Repositori ini berisi 3 API (Encryption, Defensive, Offensive) yang dideploy di Kubernetes menggunakan file YAML.

## 🗂 Struktur Proyek

```
uas1/
├── encryption-api/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
├── defensive-api/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
├── offensive-api/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
├── uaskubernetes.yaml
└── docker-compose.yml
```

---

## 🚀 Langkah Deploy

### 1. Build Image (Opsional jika image lokal)

```bash
docker build -t encryption-api ./encryption-api
docker build -t defensive-api ./defensive-api
docker build -t offensive-api ./offensive-api
```

> Jika kamu push ke Docker Hub, ubah nama imagenya menjadi `dockerhubusername/encryption-api` dan set image tersebut di YAML.

---

### 2. Deploy ke Kubernetes

```bash
kubectl apply -f uaskubernetes.yaml
```

Pastikan semua resources berhasil dibuat:

```bash
kubectl get pods
kubectl get services
```

---

### 3. Akses dari Host PC (VMware)

#### 💡 Cek IP VM:

```bash
ip a
```

Contoh IP: `192.168.100.142`

> Gunakan IP dari interface `ens33` atau yang memiliki jaringan `192.168.x.x`.

#### 💡 Cek Port NodePort:

```bash
kubectl get svc
```

Contoh:

```
encryption-api-service   NodePort   ...   5000:30080/TCP
defensive-api-service    NodePort   ...   5001:30081/TCP
offensive-api-service    NodePort   ...   5002:30082/TCP
```

---

### 4. Akses API dari Host

#### 🔎 Melalui browser:

```
http://192.168.100.142:30080     # encryption-api
http://192.168.100.142:30081     # defensive-api
http://192.168.100.142:30082     # offensive-api
```

#### 📬 Atau menggunakan `curl`:

```bash
curl http://192.168.100.142:30080/encrypt
curl http://192.168.100.142:30081/verify
curl http://192.168.100.142:30082/attack
```

---

## 🧯 Troubleshooting

* **Tidak bisa diakses dari Host**:

  * Pastikan VMware menggunakan mode **Bridged**.

  * Pastikan `ufw` di VM mengizinkan port:

    ```bash
    sudo ufw allow 30080:30082/tcp
    ```

  * Cek ulang service dengan:

    ```bash
    kubectl describe svc encryption-api-service
    ```

---

## 🧹 Pembersihan

Untuk hapus semua resource:

```bash
kubectl delete -f uaskubernetes.yaml
```

Atau hapus semua pods, services, dan deployment di namespace default:

```bash
kubectl delete pods --all
kubectl delete svc --all
kubectl delete deployment --all
```

---

Jika ingin saya bantu membuat file `uaskubernetes.yaml` ulang atau buatkan `Makefile`, tinggal bilang saja.
