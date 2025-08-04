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

ini resistant
```

---

Jika ingin saya bantu membuat file `uaskubernetes.yaml` ulang atau buatkan `Makefile`, tinggal bilang saja.

## 📊 Check the Kubernetes Dashboard

Masuk dan jelajahi dasbor Kubernetes untuk melihat aktivitas dan kondisi node K8s.

---

# Auto Scaling di Kubernetes

Auto Scaling adalah mekanisme otomatis untuk menyesuaikan jumlah pod aplikasi agar sesuai dengan kebutuhan beban kerja. Dengan Auto Scaling, aplikasi dapat tetap responsif saat beban meningkat dan menghemat sumber daya saat beban menurun.

---

## 1. Horizontal Pod Autoscaler (HPA)

HPA secara otomatis menambah atau mengurangi jumlah pod (replica) berdasarkan metrik tertentu, biasanya penggunaan CPU.

### Cara manual scale:

```sh
kubectl scale deployment tasksapp --replicas=1
```

### Membuat HPA dengan perintah:

```sh
kubectl autoscale deployment tasksapp --cpu-percent=50 --min=1 --max=5
```

* `--cpu-percent=50`: HPA akan mencoba menjaga penggunaan CPU rata-rata pod di sekitar 50%.
* `--min=1`: Jumlah minimal pod adalah 1.
* `--max=5`: Jumlah maksimal pod adalah 5.

### Testing HPA

1. Masuk ke salah satu pod untuk memberikan beban CPU tinggi secara manual:

```sh
kubectl exec -it tasksapp-564c758c68-zbkb2 -- /bin/sh
yes > /dev/null &
```

Perintah `yes > /dev/null &` akan membuat CPU sibuk.

2. Cek status pod dan HPA:

```sh
kubectl get pods -l app=tasksapp
kubectl get hpa
```

3. Jika ingin menghentikan beban CPU:

```sh
killall yes
exit
```

---

## 2. Vertical Pod Autoscaler (VPA)

VPA menyesuaikan sumber daya (CPU dan memori) yang dialokasikan ke pod secara otomatis, tanpa menambah atau mengurangi jumlah pod.

* VPA bisa diatur secara manual atau otomatis.
* Biasanya diuji melalui script yang melakukan load testing untuk melihat penyesuaian sumber daya pod.

---

### Perbedaan HPA dan VPA secara singkat:

| Aspek                | HPA (Horizontal Pod Autoscaler)                  | VPA (Vertical Pod Autoscaler)                    |
| -------------------- | ------------------------------------------------ | ------------------------------------------------ |
| Apa yang disesuaikan | Jumlah pod (replica)                             | Sumber daya pod (CPU, memori)                    |
| Pengaruh scaling     | Menambah/mengurangi jumlah pod                   | Menambah/mengurangi resource pada pod yang sama  |
| Cocok untuk          | Aplikasi yang mudah diskalakan secara horizontal | Aplikasi yang sulit diskalakan secara horizontal |
| Contoh penggunaan    | Web server dengan traffic fluktuatif             | Database atau aplikasi stateful                  |

---

# 📌 Manual Setting Resource Pod dengan `kubectl patch`

Ganti `tasksapp` dengan nama deployment kamu, dan sesuaikan nama container di patch jika berbeda.

```bash
kubectl patch deployment tasksapp --patch '{
  "spec": {
    "template": {
      "spec": {
        "containers": [{
          "name": "tasksapp-container",
          "resources": {
            "requests": {
              "cpu": "250m",
              "memory": "256Mi"
            },
            "limits": {
              "cpu": "500m",
              "memory": "512Mi"
            }
          }
        }]
      }
    }
  }
}'
```

---

# 📌 Deploy Vertical Pod Autoscaler (VPA) via `kubectl apply` dengan heredoc

Pastikan **VPA sudah terinstall** di cluster kamu (cek dengan `kubectl get pods -n kube-system | grep vpa`).

Jalankan perintah berikut untuk membuat VPA:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: tasksapp-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: tasksapp
  updatePolicy:
    updateMode: "Auto"
EOF
```

---

# 📌 Cek hasil konfigurasi

1. Cek resource di deployment:

```bash
kubectl get deployment tasksapp -o yaml | grep -A10 resources
```

2. Cek status VPA:

```bash
kubectl describe vpa tasksapp-vpa
```

---

# 📌 Penjelasan singkat

| Setting      | Fungsi                                   |
| ------------ | ---------------------------------------- |
| Manual patch | Set resource requests dan limits statis  |
| VPA          | Otomatis sesuaikan resource sesuai beban |

---

