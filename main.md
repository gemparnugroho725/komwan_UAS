Berikut adalah hasil teks yang telah **dihapus semua kata "cite" beserta pelengkapnya**:

---

### Build Image

Build aplikasi dengan perintah `docker build`:

```sh
docker build -t triandri/tasksapp-rks:1.0
```

Aplikasi juga dapat diunduh dari Docker Hub dengan perintah `docker pull`.

### Run & Test

Untuk menjalankan image secara lokal, jalankan perintah berikut:

```sh
docker network create tasksapp-net
docker run --name=mongo --rm -d --network=tasksapp-net mongo
docker run --name=tasksapp-rks --rm -p 5000:5000 -d --network=tasksapp-net triandri/tasksapp-rks:1.0
```

Cek pada browser: `localhost:5000`.

```sh
curl localhost:5000
```

---

## 🚀 Deploy Containerized App with K8s

### Components

Untuk mengelola node, diperlukan komunikasi melalui API server menggunakan file YAML atau JSON.
Kubernetes menyediakan berbagai format penulisan pada websitenya untuk komponen yang dibutuhkan.

### Deployment

Komponen deployment dibutuhkan untuk mengelola aplikasi yang berjalan di lingkungan Kubernetes, seperti manajemen replika, scale up/down, dan pembaruan.

**tasksapp.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tasksapp
  labels:
    app: tasksapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tasksapp
  template:
    metadata:
      labels:
        app: tasksapp
    spec:
      containers:
        - name: tasksapp
          image: triandri/tasksapp-rks:1.0
          ports:
            - containerPort: 5000
          imagePullPolicy: Always
```

### Create & Scale

Buat deployment `tasksapp` dengan perintah:

```sh
kubectl create -f tasksapp.yaml
```

Lihat deployment dan pod:

```sh
kubectl get deployments
kubectl get pods -o wide
```

Lakukan scale up pada deployment:

```sh
kubectl scale deployment tasksapp --replicas=3
```

Lihat pod setelah di-scale:

```sh
kubectl get pods -o wide
```

### Service

Service dengan tipe LoadBalancer digunakan untuk menyebar beban aplikasi ke beberapa pod yang telah dibuat.

**tasksapp-svc.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tasksapp-svc
spec:
  selector:
    app: tasksapp
  ports:
    - port: 8080
      targetPort: 5000
  type: LoadBalancer
```

* **port**: port yang digunakan untuk mengakses aplikasi dari luar klaster.
* **targetPort**: port yang ada pada pod.

### Run & Test

Buat service `tasksapp` dengan perintah:

```sh
kubectl create -f tasksapp-svc.yaml
```

Lihat service:

```sh
kubectl get svc tasksapp-svc
```

Akses aplikasi melalui `localhost:8080`.
Setiap permintaan akan diteruskan ke pod yang berbeda secara acak.

```sh
curl localhost:8080
```

---

## 💾 Deploy K8s Persistent Volume

Persistent Volume (PV) digunakan agar database tersimpan dalam media penyimpanan yang independen.

**mongo-pv.yaml**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
spec:
  capacity:
    storage: 200Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /Users/tss-mac-03/Documents
```

### PV Chain

Buat Persistent Volume dengan perintah:

```sh
kubectl create -f mongo-pv.yaml
```

Lihat Persistent Volume:

```sh
kubectl get pv
```

Untuk mengklaim PV, diperlukan file Persistent Volume Claim (PVC).

**mongo-pvc.yaml**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Mi
```

Buat Persistent Volume Claim:

```sh
kubectl create -f mongo-pvc.yaml
```

Lihat Persistent Volume Claim:

```sh
kubectl get pvc
```

### Deployment

Deployment MongoDB digunakan untuk mengelola database MongoDB di Kubernetes.

**mongo.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: storage
              mountPath: /data/db
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: mongo-pvc
```

Buat deployment MongoDB:

```sh
kubectl create -f mongo.yaml
```

Lihat deployment dan pod:

```sh
kubectl get deployments
kubectl get pods
```

### Services

Service MongoDB digunakan agar pod MongoDB mendapatkan alamat IP (Cluster IP) untuk berkomunikasi dengan aplikasi `tasksapp`.

**mongo-svc.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  selector:
    app: mongo
  ports:
    - port: 27017
      targetPort: 27017
```

Buat service MongoDB:

```sh
kubectl create -f mongo-svc.yaml
```

Lihat service MongoDB:

```sh
kubectl get svc mongo
```

---

## ✅ Test The Containerized App

Pengujian aplikasi dilakukan dengan melihat, menambah, mengubah, dan menghapus tugas.

* **Melihat semua task:**

  ```sh
  curl localhost:8080/tasks
  ```

* **Menambah task:**

  ```sh
  curl -X POST -H "Content-Type: application/json" -d '{"task": "Task 1"}' localhost:8080/task
  curl -X POST -H "Content-Type: application/json" -d '{"task": "Task 2"}' localhost:8080/task
  curl -X POST -H "Content-Type: application/json" -d '{"task": "Task 3"}' localhost:8080/task
  ```

* **Mengedit task:** (ganti `<id task>` dengan ID task yang sebenarnya)

  ```sh
  curl -X PUT -H "Content-Type: application/json" -d '{"task": "Task 1 Updated"}' localhost:8080/task/661bf57f6feeef7c4b172ba9
  ```

* **Menghapus task:** (ganti `<id task>` dengan ID task yang sebenarnya)

  ```sh
  curl -X DELETE localhost:8080/task/661bf5bef46597c215312a46
  ```

### Databases

Masuk ke dalam pod MongoDB untuk memeriksa data.

```sh
kubectl exec -it <nama-pod-mongo> -- /bin/bash
mongosh
```

Di dalam `mongosh`:

```sh
show dbs;
use dev;
show collections;
db.task.find();
```

### Persistent Data Test

Untuk menguji persistensi data, hapus pod database.
Kubernetes akan secara otomatis membuat pod baru karena sifat *self-healing*-nya.

```sh
kubectl delete pod <nama-pod-mongo>
```

Periksa kembali data di dalam pod MongoDB yang baru.
Data akan tetap ada, yang menunjukkan bahwa fungsi data persistent berhasil.

---

## 📊 Check the Kubernetes Dashboard

Masuk dan jelajahi dasbor Kubernetes untuk melihat aktivitas dan kondisi node K8s.

---

## असाइनमेंट Tugas 6

1. Jalankan aplikasi RESTful API yang telah dikontainerisasi (setiap kelompok) menggunakan Kubernetes di laptop masing-masing dengan menambahkan service, deployment, load balancer, dan data persistent.
2. Lakukan User Acceptance Test (UAT) pada aplikasi API yang telah dikontainerisasi dengan Kubernetes tersebut menggunakan Postman.
3. Tugas ini bersifat individu. Kumpulkan Dokumen Tugas 6 mengenai Pengelolaan Aplikasi yang Dikontainerisasi dengan Kubernetes di LMS maksimal 1 minggu setelah UAS.

---

✅ **Mau saya ubah juga ke dalam format tabel sesuai preferensi kamu?**
