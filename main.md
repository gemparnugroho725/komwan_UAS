# Pengelolaan Containerized Application Dengan Kubernetes

[cite\_start]Ini adalah ringkasan dari dokumen "Pengelolaan Containerized Application Dengan Kubernetes" oleh PoltekSSN @2024. [cite: 1, 2]

-----

## 📝 Outline

  * [cite\_start]Building Containerized App [cite: 5]
  * [cite\_start]Deploying Containerized App with Kubernetes [cite: 6]
  * [cite\_start]Deploying Kubernetes Persistent Volume [cite: 7]
  * [cite\_start]Test the Containerized App [cite: 8]
  * [cite\_start]Check the Kubernetes Dashboard [cite: 9]

-----

## 🏗️ Building Containerized App

### App Overview

[cite\_start]Aplikasi web ini dibuat untuk terhubung ke database MongoDB. [cite: 11] [cite\_start]Aplikasi ini dapat diakses secara eksternal melalui peramban web atau CLI dari mesin lokal. [cite: 12]

### App Requirements

[cite\_start]Aplikasi yang dibuat adalah aplikasi sederhana berbasis Python-Flask yang mampu berkomunikasi dengan database MongoDB. [cite: 20] [cite\_start]Aplikasi ini akan dimasukkan ke dalam kontainer Docker dan di-deploy di klaster Kubernetes. [cite: 20] [cite\_start]Tiga file yang perlu dibuat adalah `app.py`, `requirements.txt`, dan `Dockerfile`. [cite: 21] [cite\_start]Tujuan dari aplikasi ini adalah untuk memahami penggunaan pod dalam Kubernetes. [cite: 66] [cite\_start]Aplikasi ini berfungsi sebagai manajemen tugas (tasks management) untuk melihat, menambah, mengubah, dan menghapus tugas. [cite: 67]

**requirements.txt**

```text
[cite_start]Flask [cite: 36]
[cite_start]Flask-PyMongo [cite: 42]
```

**Dockerfile**

```dockerfile
[cite_start]FROM python:alpine3.7 [cite: 32]
COPY . [cite_start]/app [cite: 38]
[cite_start]WORKDIR /app [cite: 44]
[cite_start]RUN pip install -r requirements.txt [cite: 50]
[cite_start]ENV PORT 5000 [cite: 52]
[cite_start]EXPOSE 5000 [cite: 56]
[cite_start]ENTRYPOINT [ "python" ] [cite: 59]
[cite_start]CMD [ "app.py" ] [cite: 63]
```

### Build Image

Build aplikasi dengan perintah `docker build`:

```sh
[cite_start]docker build -t triandri/tasksapp-rks:1.0 . [cite: 69]
```

[cite\_start]Aplikasi juga dapat diunduh dari Docker Hub dengan perintah `docker pull`. [cite: 87]

### Run & Test

Untuk menjalankan image secara lokal, jalankan perintah berikut:

```sh
[cite_start]docker network create tasksapp-net [cite: 132]
[cite_start]docker run --name=mongo --rm -d --network=tasksapp-net mongo [cite: 133]
[cite_start]docker run --name=tasksapp-rks --rm -p 5000:5000 -d --network=tasksapp-net triandri/tasksapp-rks:1.0 [cite: 134]
```

[cite\_start]Cek pada browser: `localhost:5000`. [cite: 143]

```sh
curl localhost:5000
```

-----

## 🚀 Deploy Containerized App with K8s

### Components

[cite\_start]Untuk mengelola node, diperlukan komunikasi melalui API server menggunakan file YAML atau JSON. [cite: 149, 150] [cite\_start]Kubernetes menyediakan berbagai format penulisan pada websitenya untuk komponen yang dibutuhkan. [cite: 198]

### Deployment

[cite\_start]Komponen deployment dibutuhkan untuk mengelola aplikasi yang berjalan di lingkungan Kubernetes, seperti manajemen replika, scale up/down, dan pembaruan. [cite: 284]

**tasksapp.yaml**

```yaml
[cite_start]apiVersion: apps/v1 [cite: 286]
[cite_start]kind: Deployment [cite: 286]
[cite_start]metadata: [cite: 286]
  [cite_start]name: tasksapp [cite: 286]
  [cite_start]labels: [cite: 286]
    [cite_start]app: tasksapp [cite: 286]
[cite_start]spec: [cite: 286]
  [cite_start]replicas: 1 [cite: 286]
  [cite_start]selector: [cite: 286]
    [cite_start]matchLabels: [cite: 286]
      [cite_start]app: tasksapp [cite: 286]
  [cite_start]template: [cite: 286]
    [cite_start]metadata: [cite: 286]
      [cite_start]labels: [cite: 286]
        [cite_start]app: tasksapp [cite: 286]
    [cite_start]spec: [cite: 286]
      [cite_start]containers: [cite: 286]
        - [cite_start]name: tasksapp [cite: 286]
          [cite_start]image: triandri/tasksapp-rks:1.0 [cite: 286]
          [cite_start]ports: [cite: 286]
            - [cite_start]containerPort: 5000 [cite: 286]
          [cite_start]imagePullPolicy: Always [cite: 286]
```

### Create & Scale

Buat deployment `tasksapp` dengan perintah:

```sh
[cite_start]kubectl create -f tasksapp.yaml [cite: 292]
```

Lihat deployment dan pod:

```sh
[cite_start]kubectl get deployments [cite: 294]
[cite_start]kubectl get pods -o wide [cite: 298]
```

Lakukan scale up pada deployment:

```sh
[cite_start]kubectl scale deployment tasksapp --replicas=3 [cite: 308, 309]
```

Lihat pod setelah di-scale:

```sh
[cite_start]kubectl get pods -o wide [cite: 311]
```

### Service

[cite\_start]Service dengan tipe LoadBalancer digunakan untuk menyebar beban aplikasi ke beberapa pod yang telah dibuat. [cite: 347]

**tasksapp-svc.yaml**

```yaml
[cite_start]apiVersion: v1 [cite: 352]
[cite_start]kind: Service [cite: 356]
[cite_start]metadata: [cite: 358]
  [cite_start]name: tasksapp-svc [cite: 352]
[cite_start]spec: [cite: 361]
  [cite_start]selector: [cite: 353]
    [cite_start]app: tasksapp [cite: 364]
  [cite_start]ports: [cite: 366]
    - [cite_start]port: 8080 [cite: 368]
      [cite_start]targetPort: 5000 [cite: 371]
  [cite_start]type: LoadBalancer [cite: 372]
```

  * [cite\_start]**port**: port yang digunakan untuk mengakses aplikasi dari luar klaster. [cite: 373]
  * [cite\_start]**targetPort**: port yang ada pada pod. [cite: 374]

### Run & Test

Buat service `tasksapp` dengan perintah:

```sh
[cite_start]kubectl create -f tasksapp-svc.yaml [cite: 377]
```

Lihat service:

```sh
[cite_start]kubectl get svc tasksapp-svc [cite: 379]
```

Akses aplikasi melalui `localhost:8080`. [cite\_start]Setiap permintaan akan diteruskan ke pod yang berbeda secara acak. [cite: 398]

```sh
[cite_start]curl localhost:8080 [cite: 394, 395, 396, 397]
```

-----

## 💾 Deploy K8s Persistent Volume

[cite\_start]Persistent Volume (PV) digunakan agar database tersimpan dalam media penyimpanan yang independen. [cite: 400]

**mongo-pv.yaml**

```yaml
[cite_start]apiVersion: v1 [cite: 404]
[cite_start]kind: PersistentVolume [cite: 410]
[cite_start]metadata: [cite: 405]
  [cite_start]name: mongo-pv [cite: 413]
[cite_start]spec: [cite: 415]
  [cite_start]capacity: [cite: 417]
    [cite_start]storage: 200Mi [cite: 419]
  [cite_start]accessModes: [cite: 421]
    - [cite_start]ReadWriteOnce [cite: 423]
  [cite_start]hostPath: [cite: 425]
    [cite_start]path: /Users/tss-mac-03/Documents [cite: 427]
```

### PV Chain

Buat Persistent Volume dengan perintah:

```sh
[cite_start]kubectl create -f mongo-pv.yaml [cite: 429]
```

Lihat Persistent Volume:

```sh
[cite_start]kubectl get pv [cite: 431]
```

[cite\_start]Untuk mengklaim PV, diperlukan file Persistent Volume Claim (PVC). [cite: 444]

**mongo-pvc.yaml**

```yaml
[cite_start]apiVersion: v1 [cite: 456]
[cite_start]kind: PersistentVolumeClaim [cite: 460]
[cite_start]metadata: [cite: 462]
  [cite_start]name: mongo-pvc [cite: 465]
[cite_start]spec: [cite: 467]
  [cite_start]accessModes: [cite: 469]
    - [cite_start]ReadWriteOnce [cite: 471]
  [cite_start]resources: [cite: 473]
    [cite_start]requests: [cite: 475]
      [cite_start]storage: 200Mi [cite: 477]
```

Buat Persistent Volume Claim:

```sh
[cite_start]kubectl create -f mongo-pvc.yaml [cite: 447]
```

Lihat Persistent Volume Claim:

```sh
[cite_start]kubectl get pvc [cite: 448]
```

### Deployment

[cite\_start]Deployment MongoDB digunakan untuk mengelola database MongoDB di Kubernetes. [cite: 490]

**mongo.yaml**

```yaml
[cite_start]apiVersion: apps/v1 [cite: 480]
[cite_start]kind: Deployment [cite: 482]
[cite_start]metadata: [cite: 484]
  [cite_start]name: mongo [cite: 486]
[cite_start]spec: [cite: 488]
  [cite_start]selector: [cite: 496]
    [cite_start]matchLabels: [cite: 503]
      [cite_start]app: mongo [cite: 510]
  [cite_start]template: [cite: 517]
    [cite_start]metadata: [cite: 520]
      [cite_start]labels: [cite: 524]
        [cite_start]app: mongo [cite: 526]
    [cite_start]spec: [cite: 533]
      [cite_start]containers: [cite: 540]
        - [cite_start]name: mongo [cite: 547]
          [cite_start]image: mongo [cite: 554]
          [cite_start]ports: [cite: 556]
            - [cite_start]containerPort: 27017 [cite: 558]
          [cite_start]volumeMounts: [cite: 560]
            - [cite_start]name: storage [cite: 562]
              [cite_start]mountPath: /data/db [cite: 564]
      [cite_start]volumes: [cite: 566]
        - [cite_start]name: storage [cite: 568]
          [cite_start]persistentVolumeClaim: [cite: 570]
            [cite_start]claimName: mongo-pvc [cite: 572]
```

Buat deployment MongoDB:

```sh
[cite_start]kubectl create -f mongo.yaml [cite: 492, 493]
```

Lihat deployment dan pod:

```sh
[cite_start]kubectl get deployments [cite: 494]
[cite_start]kubectl get pods [cite: 518]
```

### Services

[cite\_start]Service MongoDB digunakan agar pod MongoDB mendapatkan alamat IP (Cluster IP) untuk berkomunikasi dengan aplikasi `tasksapp`. [cite: 574]

**mongo-svc.yaml**

```yaml
[cite_start]apiVersion: v1 [cite: 578]
[cite_start]kind: Service [cite: 580]
[cite_start]metadata: [cite: 582]
  [cite_start]name: mongo [cite: 584]
[cite_start]spec: [cite: 586]
  [cite_start]selector: [cite: 588]
    [cite_start]app: mongo [cite: 594]
  [cite_start]ports: [cite: 591]
    - [cite_start]port: 27017 [cite: 595]
      [cite_start]targetPort: 27017 [cite: 596]
```

Buat service MongoDB:

```sh
[cite_start]kubectl create -f mongo-svc.yaml [cite: 597, 598]
```

Lihat service MongoDB:

```sh
[cite_start]kubectl get svc mongo [cite: 599]
```

-----

## ✅ Test The Containerized App

[cite\_start]Pengujian aplikasi dilakukan dengan melihat, menambah, mengubah, dan menghapus tugas. [cite: 611]

  * **Melihat semua task:**

    ```sh
    [cite_start]curl localhost:8080/tasks [cite: 612]
    ```

  * **Menambah task:**

    ```sh
    [cite_start]curl -X POST -H "Content-Type: application/json" -d '{"task": "Task 1"}' localhost:8080/task [cite: 614, 615, 619]
    [cite_start]curl -X POST -H "Content-Type: application/json" -d '{"task": "Task 2"}' localhost:8080/task [cite: 614, 617, 621]
    [cite_start]curl -X POST -H "Content-Type: application/json" -d '{"task": "Task 3"}' localhost:8080/task [cite: 614, 623, 625]
    ```

  * **Mengedit task:** (ganti `<id task>` dengan ID task yang sebenarnya)

    ```sh
    [cite_start]curl -X PUT -H "Content-Type: application/json" -d '{"task": "Task 1 Updated"}' localhost:8080/task/661bf57f6feeef7c4b172ba9 [cite: 653, 655]
    ```

  * **Menghapus task:** (ganti `<id task>` dengan ID task yang sebenarnya)

    ```sh
    [cite_start]curl -X DELETE localhost:8080/task/661bf5bef46597c215312a46 [cite: 681, 682]
    ```

### Databases

Masuk ke dalam pod MongoDB untuk memeriksa data.

```sh
[cite_start]kubectl exec -it <nama-pod-mongo> -- /bin/bash [cite: 727, 728]
[cite_start]mongosh [cite: 729]
```

Di dalam `mongosh`:

```sh
[cite_start]show dbs; [cite: 730]
[cite_start]use dev; [cite: 731]
[cite_start]show collections; [cite: 733]
[cite_start]db.task.find(); [cite: 735]
```

### Persistent Data Test

Untuk menguji persistensi data, hapus pod database. [cite\_start]Kubernetes akan secara otomatis membuat pod baru karena sifat *self-healing*-nya. [cite: 778]

```sh
[cite_start]kubectl delete pod <nama-pod-mongo> [cite: 777]
```

Periksa kembali data di dalam pod MongoDB yang baru. [cite\_start]Data akan tetap ada, yang menunjukkan bahwa fungsi data persistent berhasil. [cite: 809, 810, 811, 812, 813, 814]

-----

## 📊 Check the Kubernetes Dashboard

[cite\_start]Masuk dan jelajahi dasbor Kubernetes untuk melihat aktivitas dan kondisi node K8s. [cite: 835, 836]

-----

## असाइनमेंट Tugas 6

1.  [cite\_start]Jalankan aplikasi RESTful API yang telah dikontainerisasi (setiap kelompok) menggunakan Kubernetes di laptop masing-masing dengan menambahkan service, deployment, load balancer, dan data persistent. [cite: 937] [cite\_start]Sertakan screenshot setiap tahapan (berhasil atau tidaknya). [cite: 938]
2.  [cite\_start]Lakukan User Acceptance Test (UAT) pada aplikasi API yang telah dikontainerisasi dengan Kubernetes tersebut menggunakan Postman. [cite: 939]
3.  Tugas ini bersifat individu. [cite\_start]Kumpulkan Dokumen Tugas 6 mengenai Pengelolaan Aplikasi yang Dikontainerisasi dengan Kubernetes di LMS maksimal 1 minggu setelah UAS. [cite: 940]
